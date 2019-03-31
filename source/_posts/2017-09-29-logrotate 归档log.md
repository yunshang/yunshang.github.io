---
layout: post
title: logrotate 归档log
category: tool
date: 2017-09-29
tag: 
- Tool
---

> logrotate是linux下的一个服务。它可以自动对日志进行截断（或轮循）、压缩以及删除旧的日志文件。

<!-- more -->

### logrotate 配置文件

配置文件路径 `/etc/logrotate.d`,在目录创建配置文件.

```shell
/www/project/shared/log/production.log {
    weekly
    rotate 12
    missingok
    notifempty
    compress
    dateext
    delaycompress
    copytruncate
}
```

### Cron 定时任务

语法

```
 - – – – -
 | | | | |
 | | | | +—– Day of week (0–6) (Sunday=0) or Sun, Mon, Tue,…
 | | | +———- Month (1–12) or Jan, Feb,…
 | | +————-— Day of month (1–31)
 | +——————– Hour (0–23)
 +————————- Minute (0–59)

```

cron 轮询例子

```
# /usr/sbin/logrotate -f /etc/logrotate.d/nginx  // 未到时间或者未到切割条件，强制切割
# /usr/sbin/logrotate -d -f /etc/logrotate.d/nginx // 输出切割debug信息

```

查看正在进行的 logrotate `cat /var/lib/logrotate/status`.

### 参考资料

- [http://www.jianshu.com/p/ea7c2363639c](http://www.jianshu.com/p/ea7c2363639c)

- [http://www.jianshu.com/p/0137db63b91d](http://www.jianshu.com/p/0137db63b91d)

