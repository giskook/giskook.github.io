---
title: "压测相关"
date: 2020-10-19 10:07:24+08:00
categories: work tech
---

# 压测用到的工具

## 资源

```
cat /etc/yum.repos.d/epel.repo
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.aliyun.com/epel/7/$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
baseurl=http://mirrors.aliyun.com/epel/7/$basearch/debug
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
baseurl=http://mirrors.aliyun.com/epel/7/SRPMS
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
```

## 命令

### 测量带宽
客户端 
```
iperf3 -c 192.168.0.183 -b 8000M -n 20G
```
服务端 
```
iperf3 -s
```

### 观察带宽
```
iftop
```
### 查看linux发送tcp发送
```
/proc/sys/net/ipv4/tcp_rmem (for read)
/proc/sys/net/ipv4/tcp_wmem (for write)
最小值  默认值  最大值
4096    16384   4194304
```
