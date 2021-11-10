# Docker安装

## 删除历史Docker

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

## Linux在线环境

###	安装依赖

```shell
sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
```

### 设置docker仓库

```shell
sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
```

### 设置阿里云镜像仓库

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

###	安装Docker

```shell
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

###	阿里云镜像加速器

- 访问以下网址登录自己的阿里云账户,查看自己的镜像加速地址

```html
https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
```

- 执行以下命令

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://xxxxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

##	Linux离线环境

### 下载官网安装包

(建议使用CentoOS 7 17XX 以上版本)

```html
https://download.docker.com/linux/static/stable/x86_64/
```

选择自己合适的版本,博主下载版本为**docker-18.09.9.tgz**

### 解压

上传至Linux服务器后在文件所在目录执行

```shell
tar xzvf docker-18.09.9.tgz
```

### 环境变量

复制所有docker文件至/user/bin目录下,该目录为环境变量目录

```shell
mv docker/* /usr/bin/
```

### 安装

执行Docker安装命令

```shell
dockerd
```

此处如果没有异常显示如下

```java
INFO[2021-11-01T06:54:26.836415513-07:00] Docker daemon                                 commit=039a7df9ba graphdriver(s)=overlay2 version=18.09.9
INFO[2021-11-01T06:54:26.837355719-07:00] Daemon has completed initialization          
INFO[2021-11-01T06:54:26.858023345-07:00] API listen on /var/run/docker.sock  
```

直接ctrl+c结束即可,此时Docker已经安装成功

### 设置为服务

使Docker作为系统服务启动

- 为确保服务顺利安装,请先关闭Linux中的selinux策略

```shell
vi /etc/selinux/config
```

- 修改selinux为disabled

```properties
SELINUX=disabled
```

- 执行重启命令

```shell
reboot
```

- 创建docker.service文件

```shell
vim /etc/systemd/system/docker.service
```

- docker.service文件添加以下内容

```properties
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

- 创建docker.socket文件

```shell
vim /etc/systemd/system/docker.socket
```

- docker.socket文件添加以下内容

```properties
[Unit]
Description=Docker Socket for the API
PartOf=docker.service

[Socket]
# If /var/run is not implemented as a symlink to /run, you may need to
# specify ListenStream=/var/run/docker.sock instead.
ListenStream=/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
```



5. 赋予上述两个文件执行权限

```shell
chmod +x /etc/systemd/system/docker.service
chmod +x /etc/systemd/system/docker.socket
```

7. 重新加载系统服务

```shell
systemctl daemon-reload
systemctl start docker
```

8. 设置开机自启动

```shell
systemctl enable docker
```

9. 重启Linux

```shell
reboot
```

##	启动Docker

- 执行docker启动命令

```shell
sudo systemctl start docker
```

- 测试Docker是否安装成功

```shell
sudo docker ps
```



