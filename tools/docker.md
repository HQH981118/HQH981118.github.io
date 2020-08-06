---
description: 在centos 7上安装nvidia-docker
---

# docker

## Install docker on centos 7

### First step: install docker

使用官方安装脚本自动安装，十分简易。

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

### Second step: instasll nvidia-docker

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo

sudo yum install -y nvidia-container-toolkit
```

### Third step: run docker

```bash
sudo systemctl restart docker
```

### Final step: run nvidia-docker

```text
docker run --gpus '"device=1,2"' [image_name] [CMD]
```

### Reference:

