## 背景

有一次，`光神` 在群问了个问题：

**当 close 一个 TCP 连接时，如果还有没发送完的数据在缓冲区中，内核会怎么处理？**

当时我认为，因为关闭 TCP 连接会触发四次挥手过程，而为了让四次挥手能够快速完成，应该会把发送缓冲区的数据清空，然后发送四次挥手的数据包。

带着疑问，我去查阅 Linux 源码的实现，下面就是关闭一个 TCP 连接的过程。

## 关闭 TCP 连接过程

关闭一个 TCP 连接可以使用 `close()` 系统调用，我们来分析一下当调用 `close()` 关闭一个 TCP 连接时会发生什么事情。

当调用 `close()` 系统调用时，会触发调用 `sys_close()` 内核函数，其实现如下：

```c
asmlinkage long sys_close(unsigned int fd){    
  struct file * filp;    
  struct files_struct *files = current->files;    
  ...    
  return filp_close(filp, files);    
  ...
}
```

`sys_close()` 函数最终会调用 `file_close()` 函数来关闭文件（由于在 Linux 中 socket 是一种特殊的文件），我们接着分析 `filp_close()` 函数的实现：

```c
int filp_close(struct file *filp, fl_owner_t id){    
  ...    
  fput(filp);    
  return retval;
}
void fput(struct file * file){    ...    if (atomic_dec_and_test(&file->f_count)) {        ...        if (file->f_op && file->f_op->release)            file->f_op->release(inode, file);        ...    }}
```

可以看到，最终会调用文件系统对应的 `release()` 方法来处理关闭操作。对于 socket 文件系统，`release()` 方法对应的是 `sock_close()` 函数，而 `sock_close()` 函数最终会调用 `sock_release()` 函数，所以我们来看看 `sock_release()` 函数的实现：

```c
void sock_release(struct socket *sock){    if (sock->ops)        sock->ops->release(sock);    ...}
```

`sock_release()` 函数也很简单，就是调用对应 `协议族` 的 `release()` 方法，因为 Linux 的 socket 文件系统可以支持多种协议族，比如 `INET`、`Unix Domain Socket`、`Netlink` 等。而对应 `INET协议族（网络）` 来说，这个 `release()` 方法对应的是 `inet_release()` 函数，`inet_release()` 函数实现如下：

```c
int inet_release(struct socket *sock){    struct sock *sk = sock->sk;
    if (sk) {        long timeout;        ...        timeout = 0;        if (sk->linger && !(current->flags & PF_EXITING))            timeout = sk->lingertime;        sock->sk = NULL;        sk->prot->close(sk, timeout);    }    return(0);}
```

`inet_release()` 函数最终会调用对应 `传输层（TCP或者UDP）` 的 `close()` 方法，对于 `TCP协议` 来说，`close()` 方法对应的是 `tcp_close()` 函数，`tcp_close()` 就是关闭 TCP 连接的最后站点。

由于 `tcp_close()` 函数比较复杂，我们这里只分析当发生缓冲区还有数据的情况下，内核会怎么处理缓冲区的数据。

```
void tcp_close(struct sock *sk, long timeout){    struct sk_buff *skb;    int data_was_unread = 0;
    ...    // 如果接收缓冲区有数据, 那么先情况接收缓冲区的数据    while((skb= __skb_dequeue(&sk->receive_queue)) != NULL) {        u32 len = TCP_SKB_CB(skb)->end_seq - TCP_SKB_CB(skb)->seq - skb->h.th->fin;        data_was_unread += len;        __kfree_skb(skb);    }
    ...    if (data_was_unread != 0) {                // 如果接收缓冲区有数据没有处理        tcp_set_state(sk, TCP_CLOSE);          // 把socket状态设置为TCP_CLOSE        tcp_send_active_reset(sk, GFP_KERNEL); // 发送一个reset包给对端连接    } else if (sk->linger && sk->lingertime==0) {        ...    } else if (tcp_close_state(sk)) {        tcp_send_fin(sk); // 开始发生四次挥手包    }    ...}
```

从 `tcp_close()` 函数的实现可以看出，关闭过程主要有两种情况：

- 如果接收缓冲区还有数据没有被用户处理，那么就先把接收缓冲区的数据清空，并且发送一个 reset 包给对端连接。
- 如果接收缓冲区没有数据，那么就调用 `tcp_send_fin()` 函数开始进行四次挥手过程。

四次挥手过程如下图：

![img](/Users/admin/Library/Mobile Documents/com~apple~CloudDocs/笔记/images/640.jpeg)

接下来，我们分析 `tcp_send_fin()` 函数的实现：

```c
void tcp_send_fin(struct sock *sk){    struct tcp_opt *tp = &(sk->tp_pinfo.af_tcp);    struct sk_buff *skb = skb_peek_tail(&sk->write_queue); // 发送缓冲区列表最后一个缓冲块    unsigned int mss_now;    ...    if (tp->send_head != NULL) {                         // 如果发送缓冲区不为空        TCP_SKB_CB(skb)->flags |= TCPCB_FLAG_FIN;        // 把最后一个发送缓冲块设置FIN标志        TCP_SKB_CB(skb)->end_seq++;        tp->write_seq++;    } else {                                             // 如果发送缓冲区为空        for (;;) {            skb = alloc_skb(MAX_TCP_HEADER, GFP_KERNEL); // 申请一个新的缓冲块            if (skb)                break;            current->policy |= SCHED_YIELD;            schedule();        }
        skb_reserve(skb, MAX_TCP_HEADER);        skb->csum = 0;        TCP_SKB_CB(skb)->flags = (TCPCB_FLAG_ACK | TCPCB_FLAG_FIN); // 设置FIN标志        TCP_SKB_CB(skb)->sacked = 0;
        TCP_SKB_CB(skb)->seq = tp->write_seq;        TCP_SKB_CB(skb)->end_seq = TCP_SKB_CB(skb)->seq + 1;        tcp_send_skb(sk, skb, 1, mss_now); // 发送给对端连接    }    ...}
```

在 `tcp_send_fin()` 函数我们终于找到了当发送缓冲区不为空的处理，当发送缓冲区不为空时，首先会获取发送缓冲区的最后一个缓冲块，然后把这个缓冲区的 `FIN标志位` 设置上。

所以我前面的想法是错的，当关闭一个 TCP 连接时，如果发送缓冲区还有数据没发送完，那么内核只会把发送缓冲区最后一个缓冲块设置上 `FIN标志`，而不是把发送缓冲区清空。