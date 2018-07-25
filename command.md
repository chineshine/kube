## 常用命令记录

#### 替换镜像
```
  kubectl -n <namespace> set image deployment/<deploy_name> <container_name>=<image_name>:<image_tag>
  <namespace>: 命名空间,不加命名空间默认是 default
  <deploy_name>: kubernetes deployment 的名称
  <container_name>: 部署后的容器名称
  <image_name>: 要使用的镜像名称
  <image_tag>: 镜像的标签 tag,如果是 lastest 可以省略
```
例子:
```
  kubectl -n chine set image deployment/nginx nginx=nginx:7
```

#### log/logs 命令
官方建议用 logs,将逐渐取代 log
```
  kubectl -n <namespace> logs <pod_name>
  <pod_name>: pod 的名称
```
该命令只能用于 pod
