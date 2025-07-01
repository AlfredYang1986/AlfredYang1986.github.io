# Docker Desktop 环境

## 1. 首先恢复Github的访问

在各位的 /etc/hosts 文件内部添加以下信息：

```
# Linux 系统，windows自行查阅
sudo vim /etc/hosts
```

在弹出的文件中键入如下文字
```Shell
# github
140.82.112.3 github.com
185.199.108.133 assets-cdn.github.com
185.199.109.133 assets-cdn.github.com
185.199.110.133 assets-cdn.github.com
185.199.111.133 assets-cdn.github.com
```

然后，打开浏览器，看是否能正常访问 github 
> GitHub 的访问非常的重要，很多东西都来自开源，一定要能访问

## 2. 关于下载文件 k8s 镜像文件的问题

大家大量的问题都是在下载 k8s 镜像失败。Docker Desktop为了让你付钱，背离的开源精神，强制加入了代理，并CE只能在K8S官网上下载。所以，要使用魔法打败魔法。

1. 从GitHub上弄镜像

```Shell
git clone https://github.com/AliyunContainerService/k8s-for-docker-desktop.git

git clone -b 1.32.2 https://github.com/AliyunContainerService/k8s-for-docker-desktop.git

./load_images.sh 
```
其中1.32.2替换为你本地Desktop 所显示的版本号
这样就绕过了通过软件下载镜像的过程

1. 打开Docker Desktop，然后启动k8s

2. 使用以下命令判断成功

```Shell
kubectl get nodes
```
