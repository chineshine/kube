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
