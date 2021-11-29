[TOC]



## 1.如何使用Redis来实现简单限流策略？

首先我们来看一个常见 的简单的限流策略。系统要限定用户的某个行为在指定的时间里只能允许发生 N 次，如何使用 Redis 的数据结构来实现这个限流的功能？

我们先定义这个接口，理解了这个接口的定义，读者就应该能明白我们期望达到的功能。

```
# 指定用户 user_id 的某个行为 action_key 在特定的时间内 period 只允许发生一定的次数
max_count
def is_action_allowed(user_id, action_key, period, max_count):
	return True
# 调用这个接口 , 一分钟内只允许最多回复 5 个帖子
can_reply = is_action_allowed("laoqian", "reply", 60, 5)
	if can_reply:
		do_reply()
	else:
		raise ActionThresholdOverflow()
```



## 问题：

### 1.迁移前后Redis过期时间不一致。

​	我们将`expire/pexpire/setex/psetex 命令在复制到从库的时候转换成时间戳的方式，比如expire 转成expireat命令，setex转换成set和expireat命令`

### 2.迁移前后Redis key 数量不一致

​	1、主库在做RDB快照文件的时候，发现key已经过期了，则此时不会将过期时间写在RDB中
​	2、从库在load RDB 文件到内存中的时候，发现key已经过期了，则此时不会将过期的key load进去
​	针对上述问题，***\*目前我们将以上两个步骤都改为`不忽略过期key`，过期key的删除统一由主库触发删除，然后将删除命令传送到从库中。这样key的数量就完全一致了。\****

