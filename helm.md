## helm 安装 以及 安装 nginx-ingress

#### 获取安装包
```
  wget https://kubernetes-helm.storage.googleapis.com/helm-v2.8.2-linux-amd64.tar.gz
```
解压后将文件 helm 移动到 /usr/local/bin

#### 配置 RBAC
```
  touch rbac-config.yaml
```
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
  namespace: kube-system
```
```
kubectl create -f rbac-config.yaml
```
镜像可能下不下来,请手动下载
```
  docker pull <前缀>/google_containers/tiller:v2.8.2
```
#### helm 初始化
```
  helm init --service-account tiller --tiller-image <前缀>/google_containers/tiller:v2.8.2
```

### 安装 nginx-ingress
search 找一下
```
  helm search nginx-ingress
```
fetch 下来
```
  helm fetch nginx-ingress
```
进入 values.yml 文件所在目录  
在 nginx-ingress 目录里面
```
  helm install stable/nginx-ingress --name nginx-ingress -f values.yaml --version 0.18.1 --set rbac.create=true
```
查看安装进度 :
```
  kubectl --namespace default get services -o wide -w nginx-ingress-controller
```
Note : 此处安装使用了 RBAC ,一定要加参数
```
--set rbac.create=true
```
删除已安装的包
```
  helm del --purge nginx-ingress
```
