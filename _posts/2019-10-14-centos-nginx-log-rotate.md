---
layout: post
title:  "CentOS 7 配置 Nginx 日志轮转"
date:   2019-10-14 16:40:00+0800
categories: centos nginx logrotate
---
CentOS 7 默认安装了两个工具：crontab 和 logrotate，一个负责定时任务，一个负责日志轮转。

如果使用 yum 安装 Nginx，会默认创建一份日志轮转配置文件，位于 /etc/logrotate.d/nginx，内容如下：

```
/var/log/nginx/*log {
    create 0644 nginx nginx
    daily
    rotate 10
    missingok
    notifempty
    compress
    sharedscripts
    postrotate
        /bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
    endscript
}
```

该配置指定：日志文件以天为单位轮转，最多保留十天的归档，并对归档进行压缩，允许日志文件不存在，如果日志文件为空则不进行轮转操作。详细配置参数可以查看 logrotate 文档。其中 postrotate 与 endscript 中间的这条命令的作用是通知 Nginx 重新打开日志文件。

对于该配置文件，可以使用如下命令立即执行，而不用等待定时任务的触发：

```
logrotate /etc/logrotate.d/nginx
```

每天需要被 Crontab 执行的任务脚本存放在 /etc/cron.daily/ 目录中。其中 /etc/cron.daily/logrotate 脚本被自动创建，有如下内容：

```
#!/bin/sh

/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```

该脚本会通过 logrotate 命令加载 /etc/logrotate.conf 配置文件并执行日志轮转操作。而上面提到的 /etc/logrotate.d/nginx 配置文件在 /etc/logrotate.conf 被自动引入。

简而言之：Crontab 每天会执行一遍 /etc/cron.daily/logrotate 脚本，脚本中的指令会加载 /etc/logrotate.conf 配置文件，而该配置文件会载入位于 /etc/logrotate.d/nginx 中关于 Nginx 的日志轮转配置。
