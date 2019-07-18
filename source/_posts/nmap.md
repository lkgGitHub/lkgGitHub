---
title: nmap
tags:
  - linux
date: 2019-05-29 13:26:08
---
1. 主机发现，端口扫描（指纹识别），服务和版本探测，操作系统探测，
2. 端口状态：open(开放的)， closed(关闭的)，和filtered(被过滤的) 
3. 性能：在本地网络对一个主机的默认扫描(nmap < hostname>)需要1/5秒

--exclude <host1[，host2][，host3]，...> (排除主机/网络)
如果在您指定的扫描范围有一些主机或网络不是您的目标， 那就用该选项加上以逗号分隔的列表排除它们。该列表用正常的Nmap语法， 因此它可以包括主机名，CIDR，八位字节范围等等。 当您希望扫描的网络包含执行关键任务的服务器，已知的对端口扫描反应强烈的 系统或者被其它人看管的子网时，这也许有用。
```shell
# Intense scan
nmap -T4 -A -v 192.168.1.108
# Intense scan plus UDP
nmap -sS -sU -T4 -A -v 192.168.1.108
# Intense scan, all TCP ports
nmap -p 1-65535 -T4 -A -v 192.168.1.108
# Intense scan, no ping
nmap -T4 -A -v -Pn 192.168.1.108

# Ping scan
nmap -sn 192.168.1.108
# Quick scan
nmap -T4 -F 192.168.1.108
# Quick scan plus
nmap -sV -T4 -O -F --version-light 192.168.1.108
# Quick traceroute
nmap -sn --traceroute 192.168.1.108
# Regular scan
nmap 192.168.1.108
# Slow comprehensive scan
nmap -sS -sU -T4 -A -v -PE -PP -PS80,443 -PA3389 -PU40125 -PY -g 53 --script "default or (discovery and safe)" 192.168.1.108
```