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
```
  cd /home/ngix-ingress
```
在 nginx-ingress 目录里面
```
  helm install stable/nginx-ingress --name nginx-ingress --namespace kube-system -f values.yaml --version 0.22.1 --set rbac.create=true
```
安装时候有时候仓库连不上,更新一下
```
  helm repo update
```
查看安装进度 :
```
  kubectl --namespace kube-system get services -o wide -w nginx-ingress-controller
```
Note : 此处安装使用了 RBAC ,一定要加参数
```
--set rbac.create=true
```
删除已安装的包
```
  helm del --purge nginx-ingress
```
