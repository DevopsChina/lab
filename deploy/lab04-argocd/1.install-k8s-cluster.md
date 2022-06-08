# 1. 安装工具

开始动手之前，我们来介绍两款工具，kubekey 和 k9s。

kubeykey 是 KubeSphere 团队基于 GoLang 语言开发的 kubernetes 集群部署工具，使用 KubeKey，您可以轻松、高效、灵活地单独安装和管理 Kubernetes，当然如果你部署 Kubesphere 也是分厂方便的。

k9s 是一款命令行下的 k8s 集群管理工具，可以非常有效地提高你对 k8s 的管理效率。本次分享我们对 k8s 资源的观察和操作，就是通过 k9s 来实现的。

> [kubekey release](https://github.com/kubesphere/kubekey)

> [k9s release](https://github.com/derailed/k9s)

```shell
# kubekey
curl -sfL https://get-kk.kubesphere.io | sh -

# k9s
curl -sS https://webinstall.dev/k9s | bash
```

# 2. 部署 k8s 环境

## 2.1. 生成 kubekey 配置

```shell
# 创建集群配置，集群名称为 monday
./kk create config --name monday
```

## 2.2. 修改集群配置

```yaml
# 修改配置如下
apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: monday
spec:
  hosts:
    - {
        name: tm-opsinit-01,
        address: 10.10.14.99,
        internalAddress: 10.10.14.99,
      }
  roleGroups:
    etcd:
      - tm-opsinit-01
    master:
      - tm-opsinit-01
    worker:
      - tm-opsinit-01
  controlPlaneEndpoint:
    domain: monday-api.automan.fun
    address: ""
    port: 6443
  kubernetes:
    version: v1.21.5
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: []
    insecureRegistries: []
  addons: []
```

## 2.3. 检查 docker 配置

docker daemon 默认创建的 docker0 网桥，使用 172.17.0.0 网段地址，如果你的服务器使用的 172.17.0.0 的网段，可以通过修改 docker daemon 的配置来修改 docker0 网桥的地址段，避免 地址冲突。

```json
# /etc/docker/daemon.json

{
  "log-opts": {
    "max-size": "5m",
    "max-file":"3"
  },
  "exec-opts": ["native.cgroupdriver=systemd"],
  "bip":"192.168.0.1/24"
}
```

修改完配置后，如果服务器上已安装 docker 服务，执行如下命令重载配置。如无运行 docker 服务，请忽略。

```shell
systemctl daemon-reload && systemctl restart docker
```

## 2.4. 创建集群

创建集群前，有三个点需要检查：

1. 禁用 selinux
2. 禁用防火墙
3. 安装相关系统级依赖

```shell
# 临时禁用 selinux
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

# 禁用防火墙
systemctl disable firewalld && systemctl stop firewalld && systemctl status firewalld

# 安装系统依赖
yum install socat conntrack ebtables ipset

# 创建集群
export KK_ZONE=cn
./kk create cluster -f config-monday.yaml
```

## 2.5. 安装 kubectl 客户端

> k8s 集群部署好后，我们来安装相关 kubectl 管理工具。

```shell

# 添加 k8s yum 源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装 kubectl
yum install -y kubectl

# k8s 命令行参数很多，可以通过 bash-completion 来自动补全命令
# 命令行代码补全
yum install bash-completion -y
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```

# 3. 部署 OpenELB

集群中的应用，如果要在集群外部访问，可以采用 NodePort 方式暴露服务，也可以采用 LoadBalancer 的方式。为了更好的模拟生产环境，咱们使用 LoadBalancer 的方式暴露服务。OpenLB 是由 Kubesphere 团队开发的支持裸金属服务器提供 LoadBalancer 类型服务的工具。具体功能细节就不额外讲解，大家可以自行参考官方文档。国人主导开发的工具，中文支持非常好。

> Github: `https://github.com/kubesphere/openelb`

```shell
kubectl apply -f https://raw.githubusercontent.com/openelb/openelb/master/deploy/openelb.yaml
```

## 3.1 OpenELB layer2 原理讲解

我们实验中使用 OpenLB 的 layer2 模式。这种模式需要配置一些集群主机网段的空地址，也就是说这些 IP 不能绑定物理网卡。OpenLB 提供 Eip 这个 CRD 来配置这些空 IP。为什么需要这样的 IP 呢？我们来简单讲解下 Layer2 模式的原理。

当外部有流量请求某个空 IP 时，路由器会发出 ARP 广播报文，也就是到集群服务器网段内询问，数据要发给谁。显然不会有任何一个主机响应，此时，OpenLB 的
port-manager 就会相应这个 ARP 报文。通过这种 ARP 报文欺骗的黑客手段，实现流量的劫持。剩下的步骤，就由 PortLB 来转发流量到相应的 Service 中。

PortLB 的 Layer2 模式，就是这样工作的。

```yaml
apiVersion: network.kubesphere.io/v1alpha2
kind: Eip
metadata:
  name: eip-pool
spec:
  address: 10.10.14.91-10.10.14.92
  protocol: layer2
  interface: ens192
  disable: false
```

# 4. 部署 nginx-ingress

OpenLB 可以方便地暴露 4 层协议，比如 TCP 协议等等，但在 7 层协议中的一些操作，就无能为力啦，比如卸载 https 证书，路由分发等等。

> https://kubernetes.github.io/ingress-nginx/deploy/

```shell
# 1. 应用配置清单

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml

# 2. service.metadata.annotations 中指定 OpenELB 配置

lb.kubesphere.io/v1alpha1: openelb
protocol.openelb.kubesphere.io/v1alpha1: layer2
```

> PortELB 通过这两条 annotation 信息来识别要提供服务的 Service

# 5. 部署 OpenEBS

Kubernates 集群中，对于有状态应用，需要申请 PV 支持。openEBS 支持把节点主机的文件系统映射为存储服务，这对我们的 All In One 实验来说，是个非常好的支持。

> https://openebs.io/docs/user-guides/installation

```shell
# 推荐使用 operator 部署应用
kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml
```

## 5.1 验证 hostpath

```shell
# /var/openebs/local
kubectl apply -f https://openebs.github.io/charts/examples/local-hostpath/local-hostpath-pvc.yaml
kubectl apply -f https://openebs.github.io/charts/examples/local-hostpath/local-hostpath-pod.yaml
```

# 6. 安装 Metric Server

```shell
# Installation

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

> bx509: cannot validate certificate for because it doesn't contain any IP SANs"

上述异常因证书验证未通过导致，可以配置合法域名 SSL 证书，也可通过如下方式禁用 Metric Server 证书验证.

```yaml
# Metric Server deployment 添加镜像启动参数
---
spec:
  containers:
    - args:
        # 添加如下参数
        - --kubelet-insecure-tls
```
