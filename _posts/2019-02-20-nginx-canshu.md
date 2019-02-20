---
layout: post
title:  "CentOS7定时任务详解"
categories: linux
tags:  centos7 crontab linux 工具软件  
author: cxs
---


[TOC]

### 一.Nginx并发预估
预估算法：`{（?G）*1024-system}/请求大小`

> （?G）：表示内存大小
1024：表示内存容量标准进制
system：表示系统和服务占用的额外内存和需要预留的内存
请求大小：表示静态（一般为KB）或动态（一般为MB）的请求大小

16核32G服务器，可以抗住4万多用于负载均衡的并发，最多可以抗住5-6万，跑满文件描述符。

### 二.压测工具AB
1.安装压力测试工具ab
```
[root@nginx-lua ~]# yum install httpd-tools -y
```
2.了解压测工具使用方式
```
[root@nginx-lua ~]# ab -n 200 -c 2 http://127.0.0.1/

//-n总的请求次数
//-c并发请求数
//-k是否开启长连接
```
3.参数详解
```
[root@Nginx-lua conf.d]# ab -n2000 -c2  http://127.0.0.1/index.html
...
Server Software:        nginx/1.12.2
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /index.html
Document Length:        19 bytes

Concurrency Level:      200
# 总花费总时长
Time taken for tests:   1.013 seconds
# 总请求数
Complete requests:      2000
# 请求失败数
Failed requests:        0
Write errors:           0
Total transferred:      510000 bytes
HTML transferred:       38000 bytes
# 每秒多少请求/s(总请求出/总共完成的时间)
Requests per second:    9333.23 [#/sec] (mean)
# 客户端访问服务端, 单个请求所需花费的时间
Time per request:       101.315 [ms] (mean)
# 服务端处理请求的时间
Time per request:       0.507 [ms] (mean, across all concurrent requests)
# 判断网络传输速率, 观察网络是否存在瓶颈
Transfer rate:          491.58 [Kbytes/sec] received
```
### 三.查看并发连接数和连接状态

1、查看Web服务器（Nginx Apache）的并发请求数及其TCP连接状态
```
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
netstat -n | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,"t",state[key]}'
```
返回结果一般如下
```
LAST_ACK 5 （正在等待处理的请求数）
SYN_RECV 30
ESTABLISHED 1597 （正常数据传输状态）
FIN_WAIT1 51
FIN_WAIT2 504
TIME_WAIT 1057 （处理完毕，等待超时结束的请求数）
```
其他参数说明
```
CLOSED：无连接是活动的或正在进行
LISTEN：服务器在等待进入呼叫
SYN_RECV：一个连接请求已经到达，等待确认
SYN_SENT：应用已经开始，打开一个连接
ESTABLISHED：正常数据传输状态
FIN_WAIT1：应用说它已经完成
FIN_WAIT2：另一边已同意释放
ITMED_WAIT：等待所有分组死掉
CLOSING：两边同时尝试关闭
TIME_WAIT：另一边已初始化一个释放
LAST_ACK：等待所有分组死掉
```
2、查看Nginx或apache的运行进程数
```
ps -ef | grep nginx | wc -l
ps -ef | grep httpd | wc -l
```
3、查看Web服务器进程连接数
```
netstat -antp | grep 80 | grep ESTABLISHED -c
```
