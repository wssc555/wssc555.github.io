---
title: docker compose安装过程(centos7)
tags: docker

---

# 一：准备工作

先确保已经安装好docker

# 二:安装Compose

## 1.查看docker版本：

```
docker --version
```

## 2.下载docker-compose软件：

```
[root@localhost ~]# wget https://github.com/docker/compose/releases/download/v2.16.0/docker-compose-linux-x86_64
```

## 3.改名

将其改名为docker-compose

## 4.上传文件

进入bin目录

```
cd /usr/local/bin
```

将下载的文件修改名字，并上传到服务器的/user/local/bin目录下

## 5.赋予权限

```sh
chmod +x /usr/local/bin/docker-compose
```

## 6.检查是否安装成功

```sh
docker-compose --version
```

