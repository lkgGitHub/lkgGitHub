---
title: iptables
date: 2019-05-29 14:21:05
tags: linux
---
# 简介

iptables 是一个通过控制 Linux 内核的 netfilter 模块来管理网络数据包的流动与转送的应用软件，其功能包括不仅仅包括防火墙的控制出入流量，还有端口转发等等。

引用：[http://www.zsythink.net/archives/1199]

https://wangchujiang.com/linux-command/c/iptables.html



iptables -A INPUT -m string --algo kmp --hex-string '|deadc0dedeadc0de|' -j DROPshell 命令

命令选项输入顺序

iptables 	-t 表名	 <-A/I/D/R> 规则链名 [规则号] 	<-i/o 网卡名> 		-p 协议名 			<-s 源IP/源子网> 		--sport 源端口  	<-d 目标IP/目标子网> 	 --dport 目标端口 	 -j 动作

192.168.1.112
iptables -A INPUT -s 192.168.1.112 -p tcp --dport 22 -j DROP

```shell
# 清空当前的所有规则和计数
iptables -F  # 清空所有的防火墙规则
iptables -X  # 删除用户自定义的空链
iptables -Z  # 清空计数


# 配置拒绝ssh端口连接
iptables -A INPUT -s 192.168.1.112 -p tcp --dport 22 -j DROP


# 配置白名单
iptables -A INPUT -p all -s 192.168.1.0/24 -j ACCEPT  # 允许机房内网机器可以访问
iptables -A INPUT -p all -s 192.168.140.0/24 -j ACCEPT  # 允许机房内网机器可以访问
iptables -A INPUT -p tcp -s 183.121.3.7 --dport 3380 -j ACCEPT # 允许183.121.3.7访问本机的3380端口


# 开启相应的服务端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT # 开启80端口，因为web对外都是这个端口
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT # 允许被ping
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT # 已经建立的连接得让它进来


# 列出已设置的规则  iptables -L [-t 表名] [链名]
iptables -L -t nat                # 列出 nat 上面的所有规则
iptables -L -nv 				# 查看，这个列表看起来更详细


# 清除已有规则
iptables -F INPUT  # 清空指定链 INPUT 上面的所有规则
iptables -X INPUT  # 删除指定的链，这个链必须没有被其它任何规则引用，而且这条上必须没有任何规则。 如果没有指定链名，则会删除该表中所有非内置的链。
iptables -Z INPUT  # 把指定链，或者表中的所有链上的所有计数器清零。


# 删除已添加的规则
iptables -A INPUT -s 192.168.1.5 -j DROP		# 添加一条规则
# 将所有iptables以序号标记显示，执行：
iptables -L -n --line-numbers
# 比如要删除INPUT里序号为8的规则，执行：
iptables -D INPUT 8


# 开放指定的端口
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT               #允许本地回环接口(即运行本机访问本机)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT    #允许已建立的或相关连的通行
iptables -A OUTPUT -j ACCEPT         #允许所有本机向外的访问
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    #允许访问22端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    #允许访问80端口
iptables -A INPUT -p tcp --dport 21 -j ACCEPT    #允许ftp服务的21端口
iptables -A INPUT -p tcp --dport 20 -j ACCEPT    #允许FTP服务的20端口
iptables -A INPUT -j reject       #禁止其他未允许的规则访问
iptables -A FORWARD -j REJECT     #禁止其他未允许的规则访问


# 屏蔽IP
iptables -A INPUT -p tcp -m tcp -s 192.168.0.8 -j DROP  # 屏蔽恶意主机（比如，192.168.0.8
iptables -I INPUT -s 123.45.6.7 -j DROP       #屏蔽单个IP的命令
iptables -I INPUT -s 123.0.0.0/8 -j DROP      #封整个段即从123.0.0.1到123.255.255.254的命令
iptables -I INPUT -s 124.45.0.0/16 -j DROP    #封IP段即从123.45.0.1到123.45.255.254的命令
iptables -I INPUT -s 123.45.6.0/24 -j DROP    #封IP段即从123.45.6.1到123.45.6.254的命令是


# 指定数据包出去的网络接口，只对 OUTPUT，FORWARD，POSTROUTING 三个链起作用。
iptables -A FORWARD -o eth0


# 启动网络转发规则。网210.14.67.7让内网192.168.188.0/24上网
iptables -t nat -A POSTROUTING -s 192.168.188.0/24 -j SNAT --to-source 210.14.67.127


# 端口映射。本机的 5000 端口映射到内网 虚拟机的80 端口
iptables -t nat -A PREROUTING -d 192.168.80.128 -p tcp --dport 80  -j DNAT --to-dest 192.168.80.128:5000


#字符串匹配。比如，我们要过滤所有TCP连接中的字符串test，一旦出现它我们就终止这个连接，我们可以这么做：
iptables -A INPUT -p tcp -m string --algo kmp --string "test" -j REJECT --reject-with tcp-reset
```

## 使用场景实例

- 场景一

开放 tcp 10-22/80 端口 开放 icmp 其他未被允许的端口禁止访问

存在的问题: 本机无法访问本机; 本机无法访问其他主机

- 场景二

ftp: 默认被动模式(服务器产生随机端口告诉客户端, 客户端主动连接这个端口拉取数据) vsftpd: 使 ftp 支持主动模式(客户端产生随机端口通知服务器, 服务器主动连接这个端口发送数据)

- 场景三

允许外网访问: web http -> 80/tcp; https -> 443/tcp mail smtp -> 25/tcp; smtps -> 465/tcp pop3 -> 110/tcp; pop3s -> 995/tcp imap -> 143/tcp

内部使用: file nfs -> 123/udp samba -> 137/138/139/445/tcp ftp -> 20/21/tcp remote ssh -> 22/tcp sql mysql -> 3306/tcp oracle -> 1521/tcp

- 场景四

nat 转发

- 场景五

防CC攻击

   ```shell
iptables -L -F -A -D # list flush append delete
# 场景一
iptables -I INPUT -p tcp --dport 80 -j ACCEPT # 允许 tcp 80 端口
iptables -I INPUT -p tcp --dport 10:22 -j ACCEPT # 允许 tcp 10-22 端口
iptables -I INPUT -p icmp -j ACCEPT # 允许 icmp
iptables -A INPUT -j REJECT # 添加一条规则, 不允许所有

# 优化场景一
iptables -I INPUT -i lo -j ACCEPT # 允许本机访问
iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT # 允许访问外网
iptables -I INPUT -p tcp --dport 80 -s 10.10.188.233 -j ACCEPT # 只允许固定ip访问80

# 场景二
vi /etc/vsftpd/vsftpd.conf # 使用 vsftpd 开启 ftp 主动模式
port_enable=yes
connect_from_port_20=YES
iptables -I INPUT -p tcp --dport 21 -j ACCEPT

vi /etc/vsftpd/vsftpd.conf # 建议使用 ftp 被动模式
pasv_min_port=50000
pasv_max_port=60000
iptables -I INPUT -p tcp --dport 21 -j ACCEPT
iptables -I INPUT -p tcp --dport 50000:60000 -j ACCEPT

# 还可以使用 iptables 模块追踪来自动开发对应的端口

# 场景三
iptables -I INPUT -i lo -j ACCEPT # 允许本机访问
iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT # 允许访问外网
iptables -I INPUT -s 10.10.155.0/24 -j ACCEPT # 允许内网访问
iptables -I INPUT -p tcp -m multiport --dports 80,1723 -j ACCEPT # 允许端口, 80 -> http, 1723 -> vpn
iptables -A INPUT -j REJECT # 添加一条规则, 不允许所有

iptables-save # 保存设置到配置文件

# 场景四
iptables -t nat -L # 查看 nat 配置

iptables -t nat -A POST_ROUTING -s 10.10.177.0/24 -j SNAT --to 10.10.188.232 # SNAT
vi /etc/sysconfig/network # 配置网关

iptables -t nat -A POST_ROUTING -d 10.10.188.232 -p tcp --dport 80 -j DNAT --to 10.10.177.232:80 # DNAT

#场景五
iptables -I INPUT -p tcp --syn --dport 80 -m connlimit --connlimit-above 100 -j REJECT # 限制并发连接访问数
iptables -I INPUT -m limit --limit 3/hour --limit-burst 10 -j ACCEPT # limit模块; --limit-burst 默认为5
   ```



## match

mark: mark = "0xff"
tcp: dport = "80"
state: state = "RELATED,ESTABLISHED"



## 原理

### 4个表：

filter：input, forward, output
nat：prerouting, input, output, postrouting
mangle：prerouting, input, forward, output, postrouting
raw：prerouting, output
​       优先级（由高而低）：raw –> mangle –> nat –> filter
### 5个链：
规则链名包括(也被称为五个钩子函数（hook functions）)：

INPUT链 ：处理输入数据包。
OUTPUT链 ：处理输出数据包。
PORWARD链 ：处理转发数据包。
PREROUTING链 ：用于目标地址转换（DNAT）。
POSTOUTING链 ：用于源地址转换（SNAT）。

### iptables的表与链
1. Filter表。负责过滤功能，防火墙。
Filter表示iptables的默认表，因此如果你没有自定义表，那么就默认使用filter表，它具有以下三种内建链：
INPUT链 – 处理来自外部的数据。
OUTPUT链 – 处理向外发送的数据。
FORWARD链 – 将数据转发到本机的其他网卡设备上。
2. NAT表。网络地址转换功能（network address translation）
NAT表有三种内建链：
PREROUTING链 – 处理刚到达本机并在路由转发前的数据包。它会转换数据包中的目标IP地址（destination ip address），通常用于DNAT(destination NAT)。
POSTROUTING链 – 处理即将离开本机的数据包。它会转换数据包中的源IP地址（source ip address），通常用于SNAT（source NAT）。
OUTPUT链 – 处理本机产生的数据包。
3. Mangle表。拆解报文，修改，并重新分装。
Mangle表用于指定如何处理数据包。它能改变TCP头中的QoS位。Mangle表具有5个内建链：
PREROUTING
OUTPUT
FORWARD
INPUT
POSTROUTING
4. Raw表。关闭nat表上启动的连接追踪机制
Raw表用于处理异常，它具有2个内建链：
PREROUTING chain
OUTPUT chain

### IPTABLES 规则(Rules)
匹配条件+动作 = 规则

1. 匹配条件
源地址（Source ip）， 目标地址（destination ip）
扩展条件：源端口，目标端口
2. 处理动作（target）


牢记以下三点式理解iptables规则的关键：
​    Rules包括一个条件和一个目标(target)
​    如果满足条件，就执行目标(target)中的规则或者特定值。
​    如果不满足条件，就判断下一条Rules。

目标值（Target Values）
下面是你可以在target里指定的特殊值：
​    ACCEPT – 允许防火墙接收数据包
​    DROP – 防火墙丢弃包，不给任何回应信息
​    LOG:记录到日志中，数据包传递给下一规则
​    SNAT: 源地址转换，解决内网用户用同一公网地址上网问题
​    DNAT：目标地址转换

    MASQUERADE:是snat的特殊形式，适用于动态、临时会变的ip
    REDIRECT ：重定向、映射、透明代理。
    REJECT:拒绝数据包通过，必要时会给数据发送端一个响应信息
    QUEUE – 防火墙将数据包移交到用户空间
    RETURN – 防火墙停止执行当前链中的后续Rules，并返回到调用链(the calling chain)中。

# 命令

## 语法

iptables(选项)(参数)

## 选项

```shell
-t, --table table 对指定的表 table 进行操作， table 必须是 raw， nat，filter，mangle 中的一个。如果不指定此选项，默认的是 filter 表。

# 通用匹配：源地址目标地址的匹配
-p：指定要匹配的数据包协议类型；
-s, --source [!] address[/mask] ：把指定的一个／一组地址作为源地址，按此规则进行过滤。当后面没有 mask 时，address 是一个地址，比如：192.168.1.1；当 mask 指定时，可以表示一组范围内的地址，比如：192.168.1.0/255.255.255.0。
-d, --destination [!] address[/mask] ：地址格式同上，但这里是指定地址为目的地址，按此进行过滤。
-i, --in-interface [!] <网络接口name> ：指定数据包的来自来自网络接口，比如最常见的 eth0 。注意：它只对 INPUT，FORWARD，PREROUTING 这三个链起作用。如果没有指定此选项， 说明可以来自任何一个网络接口。同前面类似，"!" 表示取反。
-o, --out-interface [!] <网络接口name> ：指定数据包出去的网络接口。只对 OUTPUT，FORWARD，POSTROUTING 三个链起作用。

# 查看管理命令
-L, --list [chain] 列出链 chain 上面的所有规则，如果没有指定链，列出表上所有链的所有规则。

# 规则管理命令
-A, --append chain rule-specification 在指定链 chain 的末尾插入指定的规则，也就是说，这条规则会被放到最后，最后才会被执行。规则是由后面的匹配来指定。
-I, --insert chain [rulenum] rule-specification 在链 chain 中的指定位置插入一条或多条规则。如果指定的规则号是1，则在链的头部插入。这也是默认的情况，如果没有指定规则号。
-D, --delete chain rule-specification -D, --delete chain rulenum 在指定的链 chain 中删除一个或多个指定规则。
-R num：Replays替换/修改第几条规则

# 链管理命令（这都是立即生效的）
-P, --policy chain target ：为指定的链 chain 设置策略 target。注意，只有内置的链才允许有策略，用户自定义的是不允许的。
-F, --flush [chain] 清空指定链 chain 上面的所有规则。如果没有指定链，清空该表上所有链的所有规则。
-N, --new-chain chain 用指定的名字创建一个新的链。
-X, --delete-chain [chain] ：删除指定的链，这个链必须没有被其它任何规则引用，而且这条上必须没有任何规则。如果没有指定链名，则会删除该表中所有非内置的链。
-E, --rename-chain old-chain new-chain ：用指定的新名字去重命名指定的链。这并不会对链内部照成任何影响。
-Z, --zero [chain] ：把指定链，或者表中的所有链上的所有计数器清零。

-j, --jump target <指定目标> ：即满足某条件时该执行什么样的动作。target 可以是内置的目标，比如 ACCEPT，也可以是用户自定义的链。
-h：显示帮助信息；
```

## 基本参数

|参数|	作用|
|--|--|
|-P|	设置默认策略:iptables -P INPUT (DROP
|-F|	清空规则链
|-L|	查看规则链
|-A|	在规则链的末尾加入新规则
|-I|	num 在规则链的头部加入新规则
|-D|	num 删除某一条规则
|-s|	匹配来源地址IP/MASK，加叹号"!"表示除这个IP外。
|-d|	匹配目标地址
|-i|	网卡名称 匹配从这块网卡流入的数据
|-o|	网卡名称 匹配从这块网卡流出的数据
|-p|	匹配协议,如tcp,udp,icmp
|--dport| num	匹配目标端口号
|--sport|num	匹配来源端口号

## iptables -h

```shell
iptables -h

iptables v1.4.7

Usage: iptables -[ACD] chain rule-specification [options]
       iptables -I chain [rulenum] rule-specification [options]
       iptables -R chain rulenum rule-specification [options]
       iptables -D chain rulenum [options]
       iptables -[LS] [chain [rulenum]] [options]
       iptables -[FZ] [chain] [options]
       iptables -[NX] chain
       iptables -E old-chain-name new-chain-name
       iptables -P chain target [options]
       iptables -h (print this help information)
```
## Commands: 

Either long or short options are allowed.

|Commands|简写|含义|中文|
|-|-|:--|-|
|--append  |-A chain| Append to chain | 在规则链的末尾加入新规则 |
|--check   |-C chain|		Check for the existence of a rule| 检查规则是否存在 |
|--delete  |-D chain| Delete matching rule from chain | 删除某一条规则 |
|--delete  |-D chain rulenum| Delete rule rulenum (1 = first) from chain | 删除某一条规则 |
|--insert  |-I chain [rulenum]| Insert in chain as rulenum (default 1=first) | 在规则链的头部加入新规则 |
|--replace |-R chain rulenum |           Replace rule rulenum (1 = first) in chain|           |
|--list    |-L [chain [rulenum]]| List the rules in a chain or all chains | 查看规则链 |
|--list-rules |-S [chain [rulenum]]|     Print the rules in a chain or all chains|     |
|--flush   |-F [chain]| Delete all rules in  chain or all chains | 清空规则链 |
|--zero    |-Z [chain [rulenum]] |       Zero counters in chain or all chains|       |
|--new     |-N chain|		Create a new user-defined chain|		|
|--delete-chain |-X [chain]|		Delete a user-defined chain|		|
|--policy  |-P chain target|      Change policy on chain to target|      |
|--rename-chain|-E old-chain new-chain|Change chain name, (moving any references)||
## Options:
|Options|简写|含义|中文描述|
|-|-|-|-|
|[!] --protocol	|-p proto|	protocol: by number or name, eg. 'tcp'| 匹配协议,如tcp,udp,icmp |
|[!] --source	|-s address[/mask][...]|      source specification| 匹配来源地址IP/MASK，加叹号"!"表示除这个IP外 |
|[!] --destination |-d address[/mask][...] |    destination specification| 匹配目标地址 |
|[!] --in-interface |-i input name[+]|     network interface name ([+] for wildcard)| 网卡名称 匹配从这块网卡流入的数据 |
|[!] --out-interface |-o output name[+]|   network interface name (+]| 网卡名称 匹配从这块网卡流出的数据 |
|[!] --fragment	|-f|		match second or further fragments only|		|
|[!] --version	|-V|		print package version.|		|
|--ipv4	|-4|		Nothing (line is ignored by ip6tables-restore)|		|
|--ipv6	|-6|		Error (line is ignored by iptables-restore)|		|
|--jump	|-j target|       target for rule (may load target extension)|       |
|--goto    |-g chain|   jump to chain with no return|   |
|--match	|-m match|        extended match (may load extension)| 扩展匹配（可以加载扩展） |
|--numeric	|-n	|	        numeric output of addresses and ports| 地址和端口的数字输出 |
|--table	|-t table|	    table to manipulate (default: 'filter')| 操作的表，默认为‘filter’ |
|--verbose	|-v|		        verbose mode|		        |
|--wait	|-w [seconds]|	maximum wait to acquire xtables lock|	|
|--wait-interval |-W [usecs]|	wait time to try to acquire|	|
|--line-numbers	||	print line numbers when listing|	|
|--exact	|-x|		expand numbers (display exact values)|		|
|--modprobe=\<command>|	|	try to insert modules using this|	|
|--set-counters PKTS BYTES|	|set the counter during insert/append||

