## 相关设置

#### 1) 关闭虚拟内存

```
  swapoff -a
```  
或永久关闭 ：    
 ```
  vi /etc/fstab
  # /dev/mapper/centos-swap swap swap  defaults   0 0
```  

#### 2) 关闭 seLinux

```
  setenforce 0
```
或永久关闭 :   
```
  vi /etc/sysconfig/selinux

  SELinux=enforcing =修改为=> SELinux=disabled
```

#### 3) 修改配置

修改配置添加两条记录  
```
  vi /etc/sysctl.conf

  net.bridge.bridge-nf-call-iptables = 1
  net.bridge.bridge-nf-call-ip6tables = 1
```
```
  sysctl -p
```

## 配置 yum 源
此处使用的是阿里的源  
地址 : https://opsx.alibaba.com/mirror
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
## 安装 k8s 相对应的包
```
  yum install -y kubelet kubeadm kubectl
```
## 启动相关服务
记得先安装docker :  
```
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```
## 初始化
```
  kubeadm init --config kubeadm.yaml
```
kubeadm.yaml 注意相关配置  
note :   
如果不使用该配置文件,则安装会使用 k8s 相关镜像,开头为:k8s.gcr.io  
需要翻墙下载,或事先下载好,  
如果名称改掉了,使用配置文件,去指定镜像  
运行成功会打印日志,类似如下 :
```
To start using your cluster, you need to run the following as a regular user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
You can now join any number of machines by running the following on each node
as root:
  kubeadm join 10.0.100.202:6443 --token thczis.64adx0imeuhu23xv --discovery-token-ca-cert-hash sha256:fa7b11bb569493fd44554aab0afe55a4c051cccc492dbdfafae6efeb6ffa80e6
```
接下来运行命令 :
```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## CNI 网络配置
不配置网络,kubectl 会起不来
以下两种任选其一 :
### FLANNER
```
mkdir -p /etc/cni/net.d/
cat <<EOF> /etc/cni/net.d/10-flannel.conf
{
    "name": "cbr0",
    "type": "flannel",
    "delegate": {
           "isDefaultGateway": true
       }
}
EOF
mkdir /usr/share/oci-umount/oci-umount.d -p
mkdir /run/flannel/
cat <<EOF> /run/flannel/subnet.env
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.1.0/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
EOF
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
```
### CALICO
```
kubectl apply -f rbac-kdd.yaml
kubectl apply -f calico.yaml
```

## 相关问题解决
### 出错如何重置
```
  kubeadm reset
```
### 配置文件位置
```
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
 init 之后,可能会一直卡住,因为某个镜像下不下来  
 使用参数指定镜像:  
 ```
 Environment="KUBELET_EXTRA_ARGS=--pod-infra-container-image=k8s.gcr.io/google_containers/pause-amd64:3.1"
```
 将镜像'k8s.gcr.io/google_containers/pause-amd64:3.1' 替换为本地已下好的镜像  
 v1.11.0 此位置变更
```
 vi /etc/sysconfig/kubelet
 KUBELET_EXTRA_ARGS=--pod-infra-container-image=registry.xonestep.com/google_containers/pause-amd64:3.1
```
### cgroup 一致
检查 docker 使用的 cgroup  
```
docker info | grep -i cgroup
```
检查 kubernetes 配置文件使用的 cgroup  
配置文件位置 :   
```
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf
## 该行
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
```
v1.11.0 默认使用的是 cgroups,且配置换了位置:
```
cat /var/lib/kubelet/kubeadm-flags.env
```
### 日志追踪查看
```
journalctl -u kubelet.service -af
# 或 土一点的方式
tail -n 100 /var/log/message
```
