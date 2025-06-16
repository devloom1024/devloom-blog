---
title: Nginx 常见错误解决方案
date: 2025-06-16 11:51:20
tags: [Nginx, 错误解决方案]
---


# 错误 `nginx: [error] invalid PID number "" in "/run/nginx.pid"` 解决方案


## 问题描述
对 nginx 执行 reload 命令时报错：

```shell
[MyHome@MyMachine ~]$ sudo nginx -s reload
nginx: [error] invalid PID number "" in "/run/nginx.pid"
```


## 解决方法

### 方法 1
重新加载配置文件 nginx.conf，然后再执行 reload

```shell
[MyHome@MyMachine ~]$ # nginx.conf 可能不在 /etc/nginx/ 下，具体视 nginx 的安装路径而定
[MyHome@MyMachine ~]$ sudo nginx -c /etc/nginx/nginx.conf
[MyHome@MyMachine ~]$ sudo nginx -s reload
```

> 较常用

### 方法 2
直接将 nginx 主进程的 PID 写入 "/run/nginx.pid"

```shell
[MyHome@MyMachine ~]$ # 下面的命令得到 nginx 主进程的PID：19386
[MyHome@MyMachine ~]$ ps -aux | grep "nginx: master process"
root     19386  0.0  0.0  70060  7308 ?        Ss   15:36   0:00 nginx: master process nginx
myname 20740  0.0  0.0 116800  1048 pts/0    S+   23:31   0:00 grep --color=auto nginx: master process
[MyHome@MyMachine ~]$ sudo echo 19386 > /run/nginx.pid
[MyHome@MyMachine ~]$ sudo nginx -s reload
```

> 如果方法1失败了，可以考虑用

### 方法 3
杀掉 nginx 的主进程，然后重启 nginx

```shell
[MyHome@MyMachine ~]$ sudo killall nginx
[MyHome@MyMachine ~]$ sudo nginx
```

> 最好别用，因为 nginx 会关闭一段时间，重启时也可能节外生枝，这可能影响到服务的使用。