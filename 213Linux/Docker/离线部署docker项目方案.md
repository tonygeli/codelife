# 离线部署docker项目方案

## 1. 项目背景



​    目前我行的信用卡项目使用docker部署，由于生产环境不能联网，因此不能通过dockerfile来进行构建并部署。本文主要介绍在离线环境下运行docker项目的方法，为项目的部署和实施提供参考。



## 2. 相关命令

（1）docker commit
该命令的作用和使用方法如下：



```
[root@localhost azkaban]# docker commit --help

Create a new image from a container's changes

Usage:	docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

  -a, --author        Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change=[]     Apply Dockerfile instruction to the created image
  --help              Print usage
  -m, --message       Commit message
  -p, --pause=true    Pause container during commit
```



docker commit 命令一般用来对容器的某个状态进行备份。



（2）docker save
该命令的作用和使用方法如下：

```
[root@localhost azkaban]# docker save --help

Save one or more images to a tar archive (streamed to STDOUT by default)

Usage:	docker save [OPTIONS] IMAGE [IMAGE...]

Save one or more images to a tar archive (streamed to STDOUT by default)

  --help             Print usage
  -o, --output       Write to a file, instead of STDOUT
```



docker save 命令主要是将容器打包成tar格式的文件，方便迁移。



（3）docker load
该命令的作用和使用方法如下：



```
[root@localhost azkaban]# docker load --help

Load an image from a tar archive or STDIN

Usage:	docker load [OPTIONS]

Load an image from a tar archive or STDIN

  --help             Print usage
  -i, --input        Read from a tar archive file, instead of STDIN
  -q, --quiet        Suppress the load output
```



docker load命令的作用是docker save的逆过程，将tar镜像文件加载到本地系统中。



（4） docker-compose up|down
主要用于启动docker整个项目。需要说明的是离线环境中，通过加载tar文件的方法将容器恢复到生产环境本地，不能使用 docker-compose start|stop命令来启动已经存在的容器,但可以用如下命令来替代：



```
[root@localhost azkaban]# docker-compose up --no-recreate
[root@localhost azkaban]#
```

## 3. 部署方案



​    离线环境下，不能联网，因此不能通过docker build去构建。但是可以先在测试环境将所有docker容器构建好，然后试运行。确认无误后，用docker save命令将容器打包成tar格式的文件，然后上传到生产环境。最后在生产环境通过docker load命令将容器文件加载到本地系统中，最后启动容器即可。



## 4. 具体实施



信用卡项目的部署步骤如下：
（1）在测试环境中，构建好容器，然后启动，并查看当前正在运行的容器



```
[root@localhost azkaban]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
3dcc2a3c9f69        02azweb             "/bin/sh -c /root/run"   16 hours ago        Up 5 seconds        0.0.0.0:8443->8443/tcp     azkaban_azweb_1
cde60815d308        03azexe             "/bin/sh -c /root/run"   16 hours ago        Up 6 seconds        0.0.0.0:12321->12321/tcp   azkaban_azexe_1
291c90c34bae        01mysql             "/bin/sh -c /root/run"   16 hours ago        Up 6 seconds        0.0.0.0:3306->3306/tcp     azkaban_myserver.local_1
[root@localhost azkaban]#
```



（2）使用docker save命令将正在运行的容器打包



```
[root@localhost azkaban]# docker save -o /root/01mysql.tar 01mysql

[root@localhost azkaban]# docker save -o /root/02azweb.tar 02azweb

[root@localhost azkaban]# docker save -o /root/03azexe.tar 03azexe
```



（3）将所有的tar文件和启动文件(docker-compose.yml)上传到生产环境(云主机)



```
[root@cdf806db98f4474 creditcard]# ls
01mysql.tar  02azweb.tar  03azexe.tar  docker-compose.yml
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]#
```



（4）使用docker load命令将镜像加载到本地



```
[root@cdf806db98f4474 creditcard]# ls
01mysql.tar  02azweb.tar  03azexe.tar  docker-compose.yml
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]# docker load -i 01mysql.tar
c8305edc6321: Loading layer [==================================================>] 132.5 MB/132.5 MB
5c42a1311e7e: Loading layer [==================================================>] 15.87 kB/15.87 kB
c3c1fd75be70: Loading layer [==================================================>] 9.728 kB/9.728 kB
cc8df16ebbaf: Loading layer [==================================================>] 4.608 kB/4.608 kB
4f9d527ff23f: Loading layer [==================================================>] 3.072 kB/3.072 kB
1b8e8c715e39: Loading layer [==================================================>]  5.12 kB/5.12 kB
e507afbee7f4: Loading layer [==================================================>] 20.45 MB/20.45 MB
39ec918b7f2f: Loading layer [==================================================>] 2.048 kB/2.048 kB
a4ed412d5891: Loading layer [==================================================>]  2.56 kB/2.56 kB
e3be9bb7b038: Loading layer [==================================================>] 1.193 MB/1.193 MB
6b76d21c819c: Loading layer [==================================================>] 898.6 kB/898.6 kB
67fcaabeec65: Loading layer [==================================================>] 3.834 MB/3.834 MB
2013683020bf: Loading layer [==================================================>] 60.06 MB/60.06 MB
de20926b4839: Loading layer [==================================================>] 5.567 MB/5.567 MB
97f46a45df06: Loading layer [==================================================>] 348.3 MB/348.3 MB
97c2201c812f: Loading layer [==================================================>] 299.4 MB/299.4 MB
f4f3be87c4b0: Loading layer [==================================================>] 3.072 kB/3.072 kB
c7d7e16e71f4: Loading layer [==================================================>] 7.168 kB/7.168 kB
0442b4ed1163: Loading layer [==================================================>]  2.56 kB/2.56 kB
68f6942b8f28: Loading layer [==================================================>]  2.56 kB/2.56 kB
[root@cdf806db98f4474 creditcard]# ====>                                        ]    512 B/2.56 kB
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
01mysql             latest              f5db485a45c1        2 weeks ago         856.3 MB
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]# docker load -i 02azweb.tar
11f4bad0bd51: Loading layer [==================================================>] 15.11 MB/15.11 MB
a59613308d7c: Loading layer [==================================================>] 2.048 kB/2.048 kB
9b739afd6122: Loading layer [==================================================>] 989.2 kB/989.2 kB
31c49ed4c3af: Loading layer [==================================================>] 8.394 MB/8.394 MB
0703bda10e6c: Loading layer [==================================================>] 9.216 kB/9.216 kB
a100de2c97a8: Loading layer [==================================================>] 4.608 kB/4.608 kB
5b6c73a56916: Loading layer [==================================================>]  2.56 kB/2.56 kB
e2785bf336a4: Loading layer [==================================================>] 4.608 kB/4.608 kB
e3ca7a8321de: Loading layer [==================================================>]  2.56 kB/2.56 kB
[root@cdf806db98f4474 creditcard]# ====>                                        ]    512 B/2.56 kB
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
02azweb             latest              5be1ca339c00        3 hours ago         582.8 MB
01mysql             latest              f5db485a45c1        2 weeks ago         856.3 MB
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]# docker load -i 03azexe.tar
97dc717cd760: Loading layer [==================================================>] 25.28 MB/25.28 MB
a66898f83caf: Loading layer [==================================================>]   234 MB/234 MB
d31e7bdbac8e: Loading layer [==================================================>] 8.128 MB/8.128 MB
da4bb674e32f: Loading layer [==================================================>] 9.338 MB/9.338 MB
6e532e578483: Loading layer [==================================================>] 12.21 MB/12.21 MB
5d15325db249: Loading layer [==================================================>] 2.048 kB/2.048 kB
744ea60090a4: Loading layer [==================================================>] 989.2 kB/989.2 kB
140f627335db: Loading layer [==================================================>] 4.608 kB/4.608 kB
3f4a866d3902: Loading layer [==================================================>]  5.12 kB/5.12 kB
b8bac96bd794: Loading layer [==================================================>] 4.608 kB/4.608 kB
4dfb19835bc7: Loading layer [==================================================>]  2.56 kB/2.56 kB
2fed8dffed35: Loading layer [==================================================>] 4.608 kB/4.608 kB
9a54867c948d: Loading layer [==================================================>]  2.56 kB/2.56 kB
[root@cdf806db98f4474 creditcard]# ====>                                        ]    512 B/2.56 kB
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
03azexe             latest              7702f5f83fdc        3 hours ago         840.9 MB
02azweb             latest              5be1ca339c00        3 hours ago         582.8 MB
01mysql             latest              f5db485a45c1        2 weeks ago         856.3 MB
```



（5）项目启动，停止



```
[root@cdf806db98f4474 creditcard]# docker-compose up
......
[root@cdf806db98f4474 creditcard]# docker-compose down
Stopping creditcard_azweb_1 ... done
Stopping creditcard_azexe_1 ... done
Stopping creditcard_myserver.local_1 ... done
Removing creditcard_azweb_1 ... done
Removing creditcard_azexe_1 ... done
Removing creditcard_myserver.local_1 ... done
[root@cdf806db98f4474 creditcard]#
[root@cdf806db98f4474 creditcard]# docker-compose up --no-recreate
......
[root@cdf806db98f4474 creditcard]#
```



（6）查看是否启动成功
使用浏览器登录azkaban-web-server的后台界面，如果无误，则部署成功了。

![alt text](http://192.168.10.226:9999/gitlab-wiki-pics/p33.PNG?ynotemdtimestamp=1618377114713)



------

## 5. 结束语

​    本文主要介绍了离线部署docker项目的方法，通过docker的两个关键命令，可以方便地将容器迁移到云主机，而且容器的启动和停止只需要一行命令可以了，非常快捷高效。

