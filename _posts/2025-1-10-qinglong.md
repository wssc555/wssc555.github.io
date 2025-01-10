---
title: Centos7安装青龙面板
tags: Centos

---
# 

## 准备环境：

1.Centos7

2.docker

3.脚本库

## 一、安装docker(如已安装可跳过)

### 1.**卸载旧版本（如果之前安装过的话）**

```shell
yum remove docker  docker-common docker-selinux docker-engine
```

### 2.安装yum工具包

```shell
$ sudo yum install -y yum-utils
```

### 3.配置仓库源

```shell
# 1. 默认使用国外源，非常非常非常慢！
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 2. 推荐用国内源，丝滑！
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo

```

### 4.安装Docker Engine

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

### 5.启动Docker

```shell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

## 二、安装青龙面板

官方地址:https://github.com/whyour/qinglong

### 1.拉取镜像

```shell
docker pull whyour/qinglong:latest
```

### 2.部署

```shell
# curl -sSL get.docker.com | sh
docker run -dit \
  -v $PWD/ql/data:/ql/data \
  # 冒号后面的 5700 为默认端口，如果设置了 QlPort, 需要跟 QlPort 保持一致
  -p 5700:5700 \
  # 部署路径非必须，比如 /test
  -e QlBaseUrl="/" \
  # 部署端口非必须，当使用 host 模式时，可以设置服务启动后的端口，默认 5700
  -e QlPort="5700" \
  --name qinglong \
  --hostname qinglong \
  --restart unless-stopped \
  whyour/qinglong:latest
```

### 3.设置防火墙端口

```shell
#放行22端口
firewall-cmd --zone=public --add-port=5700/tcp --permanent
#重载配置
firewall-cmd --reload
#查看已放行端口
firewall-cmd --zone=public --list-ports
```

其他常用命令

```shell
#如果您已经安装iptables建议先关闭
service iptables stop
#查看Firewalld状态
firewall-cmd --state
#启动firewalld
systemctl start firewalld
#设置开机启动
systemctl enable firewalld.service
```

### 4.设置云服务器安全组

进入云服务器控制台，添加一条入站规则，开放5700端口

### 5.初始化面板

在浏览器中打开{云服务器ip:5700}，进入快速开始页面，创建账号。

### 6.安装依赖

在依赖管理中配置node、python、linux依赖，添加下面依赖并选择自动拆分，等待自动安装。如果下载缓慢，可以在系统设置中配置镜像源

**Node**

```
ts-node
depend
ds
global-agent
jsdom
date-fns
ts-md5
requests
jieba
npm
node-jsencrypt
tough-cookie
-g npm
jsdom -g
js-base64
fs
json5
prettytable
cjs
tslib
@types/node
png-js
typescript
crypto-js
require
upgrade pip
axios
form-data
common
qs
dotenv
ws@7.4.3
ql
node-telegram-bot-api
crypto -g
-g typescipt
moment
```

**python**

```
pytz
typescript
ping3
httpx
canvas
PyExecJS
success
--upgrade pip
jieba
pip
requests
aiohttp
lxml
```

**linux**

```
bizMsg
gcc
magic
python-devel
bizCode
```



## 三、配置脚本库

这里使用常见的[faker2](https://github.com/shufflewzc/faker2)库

在定时任务里创建新任务，输入脚本地址，配置cron表达式。