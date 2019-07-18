---
title: redis
date: 2019-07-10 10:31:12
tags:
- redis
---
# 安装
```shell
# 安装依赖
sudo apt install make gcc
# 下载安装
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
tar xzf redis-5.0.5.tar.gz
cd redis-5.0.5
make MALLOC=libc
# 修改配置文件redis.conf，启动
src/redis-server redis.conf
```

# redis-cli 命令

```shell 
auth Cnitsec%4321
#取从位置0开始到位置2结束的3个元素。
lrange mykey 0 2
llen mykey
```

# Redis 启动时出现警告的解决办法
```shell
7283:M 12 Mar 12:13:33.749 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
7283:M 12 Mar 12:13:33.749 # Server started, Redis version 3.0.7
7283:M 12 Mar 12:13:33.749 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
7283:M 12 Mar 12:13:33.749 * The server is now ready to accept connections on port 6379
```

## 第1个警告(WARNING: The TCP backlog setting of 511 ......)解决办法
### 方法1: 临时设置生效:
> sysctl -w net.core.somaxconn = 1024
### 方法2: 永久生效: 
```shell
# 修改/etc/sysctl.conf文件，增加一行
net.core.somaxconn= 1024
# 然后执行命令
sysctl -p
```


补充:
net.core.somaxconn是linux中的一个kernel参数，表示socket监听（listen）的backlog上限。
backlog是socket的监听队列，当一个请求（request）尚未被处理或建立时，他会进入backlog。
而socket server可以一次性处理backlog中的所有请求，处理后的请求不再位于监听队列中。
当server处理请求较慢，以至于监听队列被填满后，新来的请求会被拒绝。
所以说net.core.somaxconn限制了接收新 TCP 连接侦听队列的大小。
对于一个经常处理新连接的高负载 web服务环境来说，默认的 128 太小了。大多数环境这个值建议增加到 1024 或者更多。


## 第2个警告(WARNING overcommit_memory is set to 0! ......)同样也有两个解决办法
方法1: 临时设置生效: 
> sysctl -w vm.overcommit_memory = 1
```shell
# 方法2: 永久生效: 修改/etc/sysctl.conf文件，增加一行
vm.overcommit_memory = 1
# 然后执行命令
sysctl -p
```

### 补充: 
overcommit_memory参数说明：
设置内存分配策略（可选，根据服务器的实际情况进行设置）
/proc/sys/vm/overcommit_memory
可选值：0、1、2。
0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
2， 表示内核允许分配超过所有物理内存和交换空间总和的内存
注意：redis在dump数据的时候，会fork出一个子进程，理论上child进程所占用的内存和parent是一样的，比如parent占用的内存为8G，这个时候也要同样分配8G的内存给child,如果内存无法负担，往往会造成redis服务器的down机或者IO负载过高，效率下降。所以这里比较优化的内存分配策略应该设置为 1（表示内核允许分配所有的物理内存，而不管当前的内存状态如何）。