---
layout: post
title: redisson启动异常Can't init enough connections amount
tags: 面试题
author: neal

---
* content
{:toc}
### **redisson启动异常**

**原因1**

```java
Constructor threw exception; nested exception is org.redisson.client.RedisConnectionException: Can't init enough connections amount! Only 0 from 10 were initialized. Server: /127.0.0.1:6379
```

原因，打开redis.conf文件，搜索`protected-mode`：

```java
# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
```

redis默认会开启保护模式，当没有设置密码时会不允许连接；

设置为no重启即可

**原因2**

redis配置文件没有设置密码，但是spring boot 配置文件设置了密码`password=`，把这一行注释掉就可以