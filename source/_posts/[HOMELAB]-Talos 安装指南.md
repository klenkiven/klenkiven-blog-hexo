---
title: '[HOMELAB] Talos安装指南'
sidebar:
  - toc
date: 2024-09-28 16:50:34
tags: [K8S, HomeLab]
categories: [K8S]
---

Talos 是一个不可变的Linux发行版，专门为K8S做了优化，国内暂时还没有系统的教程安装，这里给一个教程进行安装。

<!-- more -->

## 安装 Brew

国内的 Homebrew 镜像有很多，但是阿里云镜像的方便安装，可以按照下面的[教程](https://developer.aliyun.com/mirror/homebrew/)进行安装。

1. 设置镜像为阿里源，并且设置环境变量
    ```bash
    # 临时替换
    export HOMEBREW_INSTALL_FROM_API=1
    export HOMEBREW_API_DOMAIN="https://mirrors.aliyun.com/homebrew-bottles/api"
    export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.aliyun.com/homebrew/brew.git"
    export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.aliyun.com/homebrew/homebrew-core.git"
    export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.aliyun.com/homebrew/homebrew-bottles"
    ```
2. 安装 Homebrew
    ```bash
    # 从阿里云下载安装脚本并安装 Homebrew 
    git clone https://mirrors.aliyun.com/homebrew/install.git brew-install
    /bin/bash brew-install/install.sh
    rm -rf brew-install

    # 验证是否完成安装了
    brew -v
    ```
3. 设置homebrew安装源为阿里源
    ```bash
    # 永久替换

    # bash 用户
    echo 'export HOMEBREW_API_DOMAIN="https://mirrors.aliyun.com/homebrew-bottles/api"' >> ~/.bash_profile
    echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.aliyun.com/homebrew/brew.git"' >> ~/.bash_profile
    echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.aliyun.com/homebrew/homebrew-core.git"' >> ~/.bash_profile
    echo 'export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.aliyun.com/homebrew/homebrew-bottles"' >> ~/.bash_profile
    source ~/.bash_profile
    brew update

    # zsh 用户
    echo 'export HOMEBREW_API_DOMAIN="https://mirrors.aliyun.com/homebrew-bottles/api"' >> ~/.zshrc
    echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.aliyun.com/homebrew/brew.git"' >> ~/.zshrc
    echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.aliyun.com/homebrew/homebrew-core.git"' >> ~/.zshrc
    echo 'export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.aliyun.com/homebrew/homebrew-bottles"' >> ~/.zshrc
    source ~/.zshrc
    brew update
    ```
4. 将 Brew 添加到 PATH 中
    ```bash
    # 如果是 Bash 执行下面的命令
    test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
    test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
    echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.bashrc

    # 如果是 ZSH 需要执行下面的命令
    test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
    test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
    echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.zshrc
    ```
5. 卸载 brew （如果后续需要的话）
    ```bash
    # 从阿里云下载安装脚本并安装 Homebrew 
    git clone https://mirrors.aliyun.com/homebrew/install.git brew-install
    /bin/bash brew-install/uninstall.sh
    rm -rf brew-install
    ```


## 安装 Talos

使用 Brew 安装
```bash
brew install siderolabs/tap/talosctl
```

## 部署 Kubernetes 到 Docker

安装 kubectl
```bash
sudo apt install kubectl
```

在本地启动一个 K8S 集群
```bash
talosctl cluster create  \
    --registry-mirror ghcr.io=https://ghcr.nju.edu.cn \
    --registry-mirror gcr.io=https://gcr.nju.edu.cn \
    --registry-mirror registry.k8s.io=https://k8s.m.daocloud.io \
    --registry-mirror k8s.gcr.io=https://k8s-gcr.m.daocloud.io \
    --registry-mirror docker.io=https://docker.m.daocloud.io
```

```bash
talosctl cluster create  \
    --registry-mirror ghcr.io=http://172.18.0.1:5000/ghcr.io \
    --registry-mirror gcr.io=http://172.18.0.1:5000/gcr.io \
    --registry-mirror registry.k8s.io=http://172.18.0.1:5000/registry.k8s.io \
    --registry-mirror k8s.gcr.io=http://172.18.0.1:5000/k8s.gcr.io \
    --registry-mirror docker.io=http://172.18.0.1:5000/docker.io
```

检查是否安装完成
```bash
# 查看当前集群的节点信息
talosctl cluster show

# 进入节点的 dashboard，观察 READY 为 Running 即可
talosctl dashboard --nodes 10.5.0.2

# 添加当前集群的配置到配置(context)
talosctl config nodes 10.5.0.2 10.5.0.3
talosctl config info
```
