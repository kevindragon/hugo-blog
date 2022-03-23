---
title: "本地虚拟机搭建Kubernetes"
date: 2021-12-02T12:45:39+08:00
tags: [Kubernetes]
---

以下内容都是在Ubuntu20.04.3系统上完成。

## 安装Docker-ce

```shell
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

sudo apt -y install docker-ce docker-ce-cli containerd.io
```

修改 Docker 的 Cgroup Driver，编辑 ~/etc/docker/daemon.json~ (没有该文件就新建一个），添加如下参数

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors":[
    "http://docker.mirrors.ustc.edu.cn",
    "http://registry.docker-cn.com",
    "http://hub-mirror.c.163.com"
  ],
  "insecure-registries":[
    "docker.mirrors.ustc.edu.cn",
    "registry.docker-cn.com",
    "hub-mirror.c.163.com"
  ]
}
```

如果当前用户为非root用户，把当前用户加入到docker用户组。

```shell
sudo usermod -a -G docker CURRENT_USER
```

然后重启docker

```bash
sudo systemctl restart docker
```

## 允许 iptables 检查桥接流量

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

## 安装Kubernetes

```bash
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

## 关闭swap

```bash
sudo swapoff -a
```

然后修改 ~/etc/fstab~ 把 /swap.img 的行注释掉

** 获取镜像列表及版本

```bash
kubeadm config images list
```

结果如下：

```text
k8s.gcr.io/kube-apiserver:v1.22.3
k8s.gcr.io/kube-controller-manager:v1.22.3
k8s.gcr.io/kube-scheduler:v1.22.3
k8s.gcr.io/kube-proxy:v1.22.3
k8s.gcr.io/pause:3.5
k8s.gcr.io/etcd:3.5.0-0
k8s.gcr.io/coredns/coredns:v1.8.4
```

编写一个shell (pull-k8s-images.sh) 脚本从阿里云的镜像仓库拉取必须的镜像，注意不要直接拷贝上面的列表内容到 ~images~ 变量，需要把前面的 ~k8s.gcr.io/~ 去掉。并且 ~k8s.gcr.io/coredns/coredns~ 在阿里云里面的名字是就是 ~coredns~ 并没有 ~coredns~ 的中间路径。

```bash
# 下面的镜像应该去除"k8s.gcr.io/"的前缀，版本换成上面获取到的版本
images=(
    kube-apiserver:v1.22.3
    kube-controller-manager:v1.22.3
    kube-scheduler:v1.22.3
    kube-proxy:v1.22.3
    pause:3.5
    etcd:3.5.0-0
    coredns:v1.8.4
)

for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done

# coredns在阿里云里面的名字和k8s.gcr.io里面的名字不一致，需要额外修改
docker tag k8s.gcr.io/coredns:v1.8.4 k8s.gcr.io/coredns/coredns:v1.8.4
```

然后运行 ~bash pull-k8s-images.sh~

## 复制两个worker

把安装好的虚拟机复制（Worker1, Worker2），复制的时候为 ~MAC地址设定~ 选择 ~为所有网卡重新生成MAC地址~ ，并设置三个虚拟机的静态IP地址，修改文件 ~/etc/netplan/00-installer-config.yaml~ 把所有对应的IP地址修改成适当有效的IP

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.0.103/24]
      gateway4: 192.168.0.1
      nameservers:
        addresses: [192.168.0.1]
  version: 2
```

改完配置执行命令： ~sudo netplan apply~

这里需要注意的一个地方，虚拟机的网络连接方式需要选择桥接网卡，这样虚拟机就会被当作局域网内部的一个单独的机器来看待了。

> 如果你连接了vpn，很有可能换了网络连接方式之后你就连不上虚拟机了，请把vpn退出，然后再连接虚拟机。

## 设置hostname

在master节点上执行

```shell
sudo hostnamectl set-hostname master
```

分别在两个worker节点执行：

```shell
sudo hostnamectl set-hostname worker1
sudo hostnamectl set-hostname worker2
```

在master虚拟机初始化k8s环境

```shell
sudo kubeadm init
```

初始化成功之后会看到如下的信息：

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.115:6443 --token qdz06o.hwsj4fqbecad870p \
	--discovery-token-ca-cert-hash sha256:064f95e49b5d0de08c7196f21a4b3fcd15805b8ec3c2dcdeb42ed4aa6154f638
```

## 在master上安装网络插件（Calico）

```bash
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

参考Calico官方文档 [[https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-kubernetes-api-datastore-50-nodes-or-less][Install Calico networking and network policy for on-premises deployments]]

在安装的过程当中很有可能摘取镜像失败导致calico启动失败：

STATUS可能会是： ~Init:ImagePullBackOff~ ~ErrImagePull~

这时得想办法手动下载对应版本的 `calico/cni` 镜像

## 添加work节点

```bash
sudo kubeadm join 192.168.0.115:6443 --token qdz06o.hwsj4fqbecad870p --discovery-token-ca-cert-hash sha256:064f95e49b5d0de08c7196f21a4b3fcd15805b8ec3c2dcdeb42ed4aa6154f638
```

## 查看状态

在master机器上运行： `kubectl get nodes` 命令可以查看集群的所有节点

```bash
NAME            STATUS     ROLES                  AGE   VERSION
dragon-master   NotReady   control-plane,master   37m   v1.22.3
dragon-node1    NotReady   <none>                 86s   v1.22.3
dragon-node2    NotReady   <none>                 17s   v1.22.3
```

当所有节点的 STATUS 都为 Ready 的时候说明所有的节点都已经就绪，可以在集群里面部署应用了

## Autoscaling

如果运行了2个replica的nginx deployment之后，每个node上会有一个pod运行。当把node2机器关掉之后，node1机器上会自动创建一个pod运行。但是当node2重新启动之后，两个pod仍然都在node1上运行着。

## 安装Dashboard

下载v2.4.0版本的relaese包[https://github.com/kubernetes/dashboard/releases/tag/v2.4.0](https://github.com/kubernetes/dashboard/releases/tag/v2.4.0)

在master节点解压后，使用dashboard-2.4.0/aio/deploy/recommended.yaml文件进行安装。
