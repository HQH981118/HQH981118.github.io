---
description: 在centos系统上安装nvidia-docker
---

# nvidia-docker

```text
# install docker based on centos 7

## 第一步：安装docker

使用官方安装脚本自动安装，十分简易。

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```


## 第二步：安装nvidia-docker

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo

sudo yum install -y nvidia-container-toolkit
```

## 第三步：启动docker

```
sudo systemctl restart docker
```

## 启动nvidia-docker

```
docker run --gpus '"device=1,2"' [image_name] [CMD]
```

## ref:

https://www.runoob.com/docker/centos-docker-install.html

https://github.com/NVIDIA/nvidia-docker
```

