# gluster
## 安装 gluster
### 前提条件
1.两个节点  
2.有挂载盘,空盘(本例:/dev/vdb)  
3.网络连通,机器之间通过 ssh 免密互相访问  
4.关闭防火墙,否则还要配置  
```
  systemctl stop firewalld.service
  systemctl disable firewalld.service
```

###挂载盘
```
 mkfs.xfs -i size=512 /dev/vdb
 mkdir -p /data/brick1
 echo '/dev/vdb /data/brick1 xfs defaults 1 2' >> /etc/fstab
 mount -a && mount
```
### 修改 hosts 文件
```
vi /etc/hosts

<ip1> gluster1
<ip2> gluster2
```
### 安装 gluster
```
 yum install centos-release-gluster
 yum install -y glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma

 systemctl enable glusterd.service
 systemctl start glusterd.service

```
### 节点操作
```
# 添加节点
  gluster peer probe gluster2
# 删除节点
  gluster peer detach gluster2
# 节点已经断开强制删除
  gluster peer detach gluster2 force
# 查看节点状态
  gluster peer status
```
### 卷/盘(volume) 配置
两台机器上
```
mkdir -p /data/brick1/paas
```
在任意一台机器上
```
  gluster volume create paas replica 2 gluster1:/data/brick1/paas gluster2:/data/brick1/paas
```
NOTE:此处会弹出一个警告:  
官网档案给出的方案装 3个 volume 或加一个 Arbiter volume  
http://docs.gluster.org/en/latest/Administrator%20Guide/Split%20brain%20and%20ways%20to%20deal%20with%20it/  
如果机器少,无视就可以了,但生产环境还请根据官网建议装3个及以上的volume  
另外,此处 volume 必须是 replica 指定的数量的倍数  

将盘启动(任一机器上)
```
  gluster volume start paas
```
如果启动报错,日志在:
```
  /var/log/glusterfs/glusterd.log
```
查看盘的信息
```
  gluster volume info
```
查看指定盘的信息
```
   gluster volume info paas  
```
### 测试
根据官网
随意以其中一台机器为 client
```
mount -t glusterfs gluster1:/paas /mnt
  for i in `seq -w 1 100`; do cp -rp /var/log/messages /mnt/copy-test-$i; done
```
check
```
ls -lA /mnt/copy* | wc -l
```
每个节点都将看到100个文件
```
 ls -lA /data/brick1/paas/copy*
```
如果没有复制(即指定 --replica),每个节点都将会只有50个文件

### 官网相关地址
github:
```
  https://github.com/gluster
```
文档:
```
  https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/
```


# heketi
假设此时操作都是在 ``<ip1>``
## 安装
```
yum -y install heketi heketi-client
```
### 配置 heketi.json
此处配置文件的位置放在 `/etc/heketi/heketi.json`  
[heketi.json](heketi.json)  
参考地址:
`https://github.com/heketi/heketi/blob/master/docs/admin/server.md`
注意: 此时如果 /dcos/heketi 下有 heketi.db ,删掉

### 配置 ssh
因为配置文件类指定的是 ssh ,所有要配置 ssh
```
mkdir /etc/heketi
ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
#chown heketi:heketi /etc/heketi/heketi_key*
```
拷贝密钥
```
ssh-copy-id -i /etc/heketi/heketi_key.pub root@<ip2>
```
上面步骤操作等同于
```
scp /etc/heketi/heketi_key.pub root@<ip2>:/tmp
ssh root@<ip2>
cat /tmp/heketi_key.pub >> /root/.ssh/authorized_keys
```

### 启动
```
  systemctl enable heketi && systemctl start heketi
```
用`systemctl`本作者操作未成功,直接使用命令启动
```
  nohup heketi --config /etc/heketi/Heketi.json &
```

### 访问测试
```
  curl http://localhost:10001/hello
```
### heketi-cli
1)创建 cluster  
用户和密钥参考 heketi.json 文件
```
  heketi-cli  --user admin --server http://<ip1>:10001 --secret admin123 --json cluster create
```
 生成 结果如下:
 ```
 {"id":"8f33e5fab8e337425774edd70addb480","nodes":[],"volumes":[],"block":true,"file":true,"blockvolumes":[]}
 ```
 上面 id 是接下来要使用的,忘了也没关系,可查找
 ```
   heketi-cli cluster list
 ```
 由于默认请求的是 8080 端口,且需要指定用户和密钥
 ```
 heketi-cli cluster list --server  http://<ip1>:10001 --user admin --secret admin123
 ```
2)添加节点
 ```
 heketi-cli --json node add --cluster "8f33e5fab8e337425774edd70addb480"  --management-host-name <ip1> --storage-host-name <ip1> --zone 1
 ```
 照此命令添加ip2的节点  
 注意此处 `<ip1>` 必须是 ip地址的数字,否则对接kubernetes 报错  
 3)在每台节点上添加设备
 ```
 heketi-cli --server http://<ip1>:10001 --user admin --secret admin123 --json device add --name="/dev/vdb" --node ="a1423d328d609acc1ddssadc0b6ab4889"
 ```
 `--node` 添加每个节点时生成的,每个节点各不同

 或将步骤 1) 2) 3) 结合为配置文件 [topology.json](topology.json)  
 注意修改 ip 地址
 ```
   vi /etc/heketi/topology.json
 ```
以配置文件方式添加
 ```
 heketi-cli --server http://<ip1>:10001 --user admin --secret 123456 topology load --json=/etc/heketi/topology.json
 ```
 容器方式:  
 `<container_id>` -> 容器的 ID
 ```
 docker cp topology.json <container_id>:/etc/heketi/
    docker exec <container_id> heketi-cli --server http://<ip1>:10001 --user admin --secret admin123 topology load --json=/etc/heketi/topology.json

 ```
 kubernetes 方式:
 ```
 kubectl cp topology.json <kubernetes_heketi_pod>:/etc/heketi/ -n kube-system
    kubectl exec -it <kubernetes_heketi_pod> -n kube-system heketi-cli topology load -- --json=/etc/heketi/topology.json --server http://<ip1>:10001 --user admin --secret admin123
 ```

### 创建 volume
 命令详情请参照:
 ```
   heketi-cli volume create --help

--size 单位为 Gi
--replica 副本数,不指定默认为3
--server heketi 服务,默认是 http://localhost:8080 ,可以通过环境变量 HEKETI_CLI_SERVER 设置
--cluster 可以同 "," 指定多个
 ```
直接方式:
 ```
 heketi-cli --server http://<ip1>:10001 --user admin --secret admin123 volume create --size=100 --replica=2 --clusters=<cluster-id>
 ```
 容器方式:
 ```
 docker exec 容器ID heketi-cli --server http://<ip1>:10001 --user admin --secret admin123 volume create --size=3 --replica=2 --clusters=<cluster-id>
 ```
 kubernetes 方式:
 ```
 kubectl exec -it <heketi-pod> -n <namespace> heketi-cli  --server http://<ip1>:10001 --user admin --secret 123456 volume create --size=100 --replica=2 --clusters=d691b29a06374c4da7e94bba71d027bf
 ```
