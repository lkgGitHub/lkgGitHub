---
title: RedisBloom 布隆过滤器
date: 2019-07-18 14:04:20
tags: redis
categories: 
---

# RedisBloom 安装 
参考：https://oss.redislabs.com/redisbloom/ 
```shell
# branch参数指定克隆的tag。
git clone --branch v2.0.2 https://github.com/RedisBloom/RedisBloom.git 
cd RedisBloom
make
# loadmodule 加载自定义module，也可以在redis.conf中
src/redis-server --loadmodule .RedisBloom/redisbloom.so
```
# RedisBloom命令
```shell
# 设置
BF.RESERVE {key} {error_rate} {capacity}
# 添加
BF.ADD {key} {item}
BF.MADD {key} {item} [item...]
# 检验是否存在
BF.EXISTS {key} {item}
BF.MEXISTS {key} {item} [item...]
```