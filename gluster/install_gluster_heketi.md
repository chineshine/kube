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
本作者找到启动失败的原因了,SSH 问题,建议检查一下
```
  ssh -i /etc/heketi/heketi_key
```
注意 heketi.json 中,ssh 配置那块,port 和 fstab 一定不要不管,否则后面 heketi起不来,而且,heketi-cli 也用不了
### 访问测试
```
  curl http://localhost:10001/hello
```
### heketi-cli
先创建三个环境变量,在 `/etc/profile.d` 目录创建自定义环境变量文件,如
```
  vi /etc/profile.d/env.sh

  export HEKETI_CLI_SERVER=http://172.21.1.160:10001
  export HEKETI_CLI_USER=admin
  export HEKETI_CLI_KEY=admin123
```
这三个环境变量分别对应 `heketi-cli` 的三个参数:
```
  参数的值都配置在 heketi.json 文件中
  --server # 服务器地址 如 http://<ip1>:10001
  --user   # 用户 如 admin
  --secret # 密钥 如 admin123
```
1)创建 cluster  
用户和密钥参考 heketi.json 文件
`--json` 参数代表以 json 格式输出
```
  heketi-cli cluster create --json
```
 生成结果如下:
 ```
 {"id":"8f33e5fab8e337425774edd70addb480","nodes":[],"volumes":[],"block":true,"file":true,"blockvolumes":[]}
 ```
 上面 id 是接下来要使用的,忘了也没关系,可查找
 ```
   heketi-cli cluster list
 ```
 此处已经配置环境变量,故而无需指定三个参数,如果不指定,由于默认请求的是 8080 端口,且需要指定用户和密钥
 ```
 heketi-cli cluster list --server  http://<ip1>:10001 --user admin --secret admin123
 ```

2)添加节点
 ```
 heketi-cli node add --cluster "8f33e5fab8e337425774edd70addb480"  --management-host-name <ip1> --storage-host-name <ip1> --zone 1
 ```
 照此命令添加ip2的节点,注意此处 `<ip1>` 必须是 ip地址的数字,否则对接kubernetes 报错  
 3)在每台节点上添加设备
 ```
 heketi-cli device add --name=/dev/vdb --node ="a1423d328d609acc1ddssadc0b6ab4889"
 ```
 `--node` 是添加每个节点时生成的值,每个节点各不同

 上述步骤 1) 2) 3) 可结合为配置文件  
  [topology.json](topology.json)  
 注意修改 ip 地址
 ```
   vi /etc/heketi/topology.json
 ```
以配置文件方式添加
 ```
 heketi-cli topology load --json=/etc/heketi/topology.json
 ```
@Note: 此处由于安装 gluster 被官方文档骗了一下,`mount`了目录`/data/brick1`,将此目录 `umount`掉  
```
  gluster volume stop paas
  gluster volume delete paas
```
先停掉正在运行的 volume,并删除,然后 `umount`
```
umount /data/brick1
```
再修改 `/etc/fstab`,将向前添加的注掉
```
  vi /etc/fstab  

  #/dev/vdb /data/brick1 xfs defaults 1 2
```
然后添加 device,由于此前被官方骗了,格式化为 xfs  
故需要添加参数`--destroy-existing-data`
```
  heketi-cli device add --name=/dev/vdb --node =a1423d328d609acc1ddssadc0b6ab4889 --destroy-existing-data
```
参数`--destroy-existing-data` 官方给出此操作是危险的,由于是第一次安装,反正没有数据,无所谓了

### 创建 volume
 命令详情请参照:
 ```
   heketi-cli volume create --help

--size     # 单位为 Gi
--replica  # 副本数,不指定默认为3
--server   # heketi 服务
--cluster  # 集群id  可以同 "," 指定多个
 ```
直接方式:
 ```
 heketi-cli volume create --size=100 --replica=2 --clusters=<cluster-id>
 ```
