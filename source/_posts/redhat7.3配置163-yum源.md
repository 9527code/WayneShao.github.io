---
title: redhat7.3配置163 yum源
tags:
  - redhat
  - yum源
abbrlink: 12206
date: 2017-01-15 11:53:25
categories:
  - 踩坑笔记
---
redhat 的更新包只对注册的用户生效，所以我们需要自己手动更改成CentOS 的更新包，CentOS几乎和redhat是一样的，所以无需担心软件包是否可安装，安装之后是否有问题。
<!-- more -->
## 删除redhat原有的yum
首先删除redhat原有的yum ，因为redhat 原本的yum 没有注册为redhat用户是用不了的。
```bash
rpm -aq|grep yum|xargs rpm -e --nodeps 
rpm -aq|grep python-iniparse|xargs rpm -e --nodeps
```
## 下载163的yum 安装包
```bash
wget http://mirrors.163.com/centos/7.3.1611/os/x86_64/Packages/yum-3.4.3-150.el7.centos.noarch.rpm
wget http://mirrors.163.com/centos/7.3.1611/os/x86_64/Packages/python-iniparse-0.4-9.el7.noarch.rpm
wget http://mirrors.163.com/centos/7.3.1611/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7.3.1611/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-40.el7.noarch.rpm
```
### 安装下载的rpm包
```bash
rpm -ivh *.rpm
```
### 创建文件/etc/yum.repos.d/rhel-debuginfo.repo并写入
```
[base]
name=CentOS-$releasever - Base
baseurl=http://mirrors.163.com/centos/7.3.1611/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/7.3.1611/os/x86_64/RPM-GPG-KEY-CentOS-7
 
 
#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=http://mirrors.163.com/centos/7.3.1611/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/7.3.1611/os/x86_64/RPM-GPG-KEY-CentOS-7
 
 
[extras]
name=CentOS-$releasever - Extras
baseurl=http://mirrors.163.com/centos/7.3.1611/extras//$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/7.3.1611/os/x86_64/RPM-GPG-KEY-CentOS-7
 
[centosplus]
name=CentOS-$releasever - Plus
baseurl=http://mirrors.163.com/centos/7.3.1611/centosplus//$basearch/
gpgcheck=1
enabled=0
```
### yum clean all
```
yum clean all
```
### yum update 测试。
```
yum update
```
### 安装 epel 源
```
yum install epel-release
```