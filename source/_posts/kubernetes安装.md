---
title: 使用kubeadm安装Kubernetes
tags: devops
categories: 运维
---
因为业务要求，需要搭建一个容器平台，选择的Kubernetes+Docker搭建集群。
Kubernetes有多种安装方式，因为使用的不是Google Cloud和AWS，使用的是阿里云，所以采用kubeadm安装Kubernetes集群。
* 准备工作
* 安装docker
* 安装kubeadm、kubelet、kubectl
* 搭建集群
---
<!-- more -->
## 准备工作
测试机器： CentOS7.5 （2核4G），3台机器，1台master，2台nodes

### 关闭swap
从Kubernetes 1.8开始如果Node上开启了swap，kubelet会启动失败。所以如果服务器是专门用作k8s Node节点的话需要将系统的swap关闭。在测试环境中可以为kubelet加上启动参数`--fail-swap-on=false`
``` bash
$ swapoff -a
```

## 安装docker

### 安装docker
``` bash
$ sudo yum install -y docker
$ sudo systemctl enable docker && sudo systemctl start docker
```

### 配置docker代理 
因为后续kubeadm需要下载google镜像，所以为docker配置代理。
创建`docker.service.d`文件夹
``` bash
$ mkdir -p /etc/systemd/system/docker.service.d
```
创建一个文件`/etc/systemd/system/docker.service.d/http-proxy.conf`，包含`HTTP_PROXY`环境变量
``` bash
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/"
```
如果有局域网或者国内的镜像仓库，我们还需要使用`NO_PROXY`变量声明一下，比如国内的daocloud.io放有镜像：
``` bash
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,daocloud.io"
```
然后重启docker即可

## 安装kubeadm、kubelet、kubectl

### 使用yum安装
添加google官方的yum repo
``` bash
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
因为国内无法访问google的yum repo，可以配置阿里云的yum repo，但是阿里云的Kubernetes版本只到1.7.5，不能安装1.7.5以上的Kubernetes版本。
注意：此坑极大，之前使用阿里云的yum源，死活安装不了Kubernetes1.9.1。后来通过先升级kubeadm版本，然后使用kubeadm upgrade升级版本。
``` bash
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
然后安装kubelet、kubeadm、kubectl
``` bash 
$ sudo yum install -y kubelet kubeadm kubectl
$ sudo systemctl enable kubelet && sudo systemctl start kubelet
```

### 手动安装
由于google和阿里云yum repo的坑，可以采取手动安装。
先将kubeadm、kubelet、kubectl下载
```
https://storage.googleapis.com/kubernetes-release/release/v1.9.1/bin/linux/amd64/kubeadm
https://storage.googleapis.com/kubernetes-release/release/v1.9.1/bin/linux/amd64/kubelet
https://storage.googleapis.com/kubernetes-release/release/v1.9.1/bin/linux/amd64/kubectl
```
然后将kubeadm、kubelet、kubectl文件上传到节点，复制到/usr/bin目录下
``` bash
$ sudo mv kube* /usr/bin
```
并为kubelet制作systemd启动服务。
创建`/etc/systemd/system/kubelet.service`文件
``` bash
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=http://kubernetes.io/docs/

[Service]
ExecStart=/usr/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```
创建`/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`文件，配置kubelet启动参数
```
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--kubeconfig=/etc/kubernetes/kubelet.conf --require-kubeconfig=true"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt"
Environment="KUBELET_CADVISOR_ARGS=--cadvisor-port=0"
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CGROUP_ARGS $KUBELET_EXTRA_ARGS
```

## 搭建集群

### master节点搭建
使用kubeadm初始化master节点
``` bash
kubeadm init --kubernetes-version=v1.9.0 --pod-network-cidr==10.244.0.0/16
```
然后kubeadm会执行一系列操作，下载镜像，安装etcd和其他控件等等。
注意，初始化master节点时一定需要关闭代理，否则会出现网络连接错误
``` bash
unset http_proxy
unset https_proxy
```
如果安装出现错误，可以查看kubelet日志以及服务日志
注意：日志这块感觉不是很明确，很难确认问题
``` bash 
sudo kubelet logs
```
``` bash
journalctl -xe
```
{% asset_img Kubernetes安装-1.png Kubernetes安装 %}
{% asset_img Kubernetes安装-2.png Kubernetes安装 %}
安装完成后，我们还不能使用kubectl来控制集群，要让kubectl可用，还要执行以下步骤
``` bash
# 对于非root用户
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 对于root用户
$ export KUBECONFIG=/etc/kubernetes/admin.conf
# 也可以直接放到~/.bash_profile
$ echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
```

### 加入nodes
部署完master节点后，就可以增加一些node到我们的集群里
ssh到我们的node节点，执行安装后出现的`kubeadm join`命令
``` bash
$ kubeadm join --token 309e0c.36f306a39b0ac005 172.16.186.72:6443 --discovery-token-ca-cert-hash sha256:3c51d780d36132282d5d3fb0da972619fb7f7b0f2d8fd71c8e89f080cc14433f
```
然后，我们去master上输入`kubectl get nodes`查看就可以看到增加的节点

## 参考链接：
https://www.kubernetes.org.cn/3357.html
http://tonybai.com/2017/01/24/explore-kubernetes-cluster-installed-by-kubeadm/
https://testerhome.com/topics/8668

