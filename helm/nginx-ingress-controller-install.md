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
