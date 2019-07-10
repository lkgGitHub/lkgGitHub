---
title: windows zip包安装
date: 2019-07-10 10:26:12
tags:
- mysql
---
# windows zip包安装
```shell
.\mysqld.exe --initialize-insecure --user=mysql
 .\mysqld.exe install
net start mysql
mysql -ruoo -p
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'lkg123456' WITH GRANT OPTION;
flush privileges;

show variables like '%character%';

--defaults-file="C:\Software\mysql-5.7.23-winx64\my-default.ini"


# window 删除
4》运行“regedit”文件，打开注册表。
删除HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\MySQL文件夹
删除HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\MySQL文件夹。
删除HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\MySQL的文件夹。
如果没有相应的文件夹，就不用删除了。
```

## my-default.ini样例
```
[mysqld]
port=3306
#这一句解决有IPV6协议的计算机上默认采用IPV6协议导致无法从程序连接数据库的问题
bind-address=127.0.0.1
# 允许最大连接数
max_connections = 200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server = utf8mb4
# 创建新表时将使用的默认存储引擎
default-storage-engine = INNODB

[client]
# 设置mysql客户端默认字符集
default-character-set = utf8mb4
```