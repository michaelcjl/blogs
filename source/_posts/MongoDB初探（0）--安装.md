---
title: MongoDB初探（0）--安装
date: 
tags: MongoDB
categories: Nosql

---

# MongoDB初探（0）--安装
早就听说MongoDB的大名了，苦于要学的东西太多，最近才抽空看一下MongoDB实战，顺便记录一下吧。

安装环境：ubuntu12.04LTS
1.命令行导入MongoDB public GPG Key:

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

 2.生成MongoDB列表文件
```
echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release -sc)"
/mongodb-org/3.0 multiverse"|sudotee/etc/apt/sources.list.d/mongodb-org-3.0.list
```
3.重载本地包数据库

    sudo apt-get update
    
4.安装MongoDB服务器端

     sudo apt-get install mongodb-10gen

 安装完成后，MongoDB服务器会自动启动，我们检查MongoDB服务器程序
 > * 检查MongoDB服务器系统进程
  ps -aux|grep mongo
 > * 通过启动命令检查MongoDB服务器状态
  netstat -nlt|grep 27017
> *  通过启动命令检查MongoDB服务器状态
sudo /etc/init.d/mongodb status
> * 通过系统服务检查MongoDB服务器状态
sudo service mongodb status

此外还可以通过web的控制台，查看MongoDB服务器的状态。在浏览器输入 http://ip:28017 ，就可以打开通过web的控制台了。

5.通过命令行客户端访问MongoDB
安装MongoDB服务器，会自动地一起安装MongoDB命令行客户端程序。在本机输入mongo命令就可以启动，客户端程序访问MongoDB服务器。

    mongo

 MongoDB服务器，默认情况下是允许外部访问的。这样单节的MongoDB，我们已经成功地安装在Ubuntu系统中。



*本人也是初学，如有不足之处，还望大家指正，希望和大家一起交流学习，共同进步。*



