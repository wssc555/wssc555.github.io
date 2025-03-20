---
title: docker安装过程(centos7)
tags: docker

---



# 一、安装环境介绍

安装环境：Linux CentOS 7

本安装教程参考Docker官方文档，地址如下：https://docs.docker.com/engine/install/centos/

其他操作系统版本也可以参考官方的其他安装版本文档。

# 二、卸载旧版docker

首先如果系统中已经存在旧的Docker，则先卸载：

```sh
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine \
    docker-selinux
```

# 三、配置docker的yum库

首先要安装一个yum工具

```sh
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

安装成功后，执行命令，配置Docker的yum源（已更新为阿里云源）：

```sh
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```

更新yum，建立缓存

```sh
sudo yum makecache fast
```

# 四、安装docker

最后，执行命令，安装Docker

```sh
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

# 五、启动和校验

```sh
# 启动Docker
systemctl start docker
# 停止Docker
systemctl stop docker
# 重启
systemctl restart docker
# 设置开机自启
systemctl enable docker
# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps
```

# 六、配置镜像加速

镜像地址可能会变更，如果失效可以百度找最新的docker镜像。

配置镜像步骤如下：

```sh
# 创建目录
mkdir -p /etc/docker
# 复制内容
tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "http://hub-mirror.c.163.com",
        "https://mirrors.tuna.tsinghua.edu.cn",
        "http://mirrors.sohu.com",
        "https://ustc-edu-cn.mirror.aliyuncs.com",
        "https://ccr.ccs.tencentyun.com",
        "https://docker.m.daocloud.io",
        "https://docker.awsl9527.cn"
    ]
}
EOF
# 重新加载配置
systemctl daemon-reload
# 重启Docker
systemctl restart docker
```

