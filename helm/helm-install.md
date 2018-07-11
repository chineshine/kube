## helm 安装 以及 安装 nginx-ingress

#### 获取安装包
```
  wget https://kubernetes-helm.storage.googleapis.com/helm-v2.8.2-linux-amd64.tar.gz
```
解压后将文件 helm 移动到 /usr/local/bin
```
  mv helm /usr/local/bin
```

#### 配置 RBAC
```
kubectl create -f rbac-config.yaml
```
镜像可能下不下来,请手动下载
```
  docker pull registry.xonestep.com/google_containers/tiller:v2.8.2
```
#### helm 初始化
```
  helm init --service-account tiller --tiller-image registry.xonestep.com/google_containers/tiller:v2.8.2
```
