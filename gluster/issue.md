### peer probe: failed: Probe returned with Transport endpoint is not connected
1.确保各个机器上是否都安装了 gluster  
2.确保各个机器是否都关闭了防火墙
```
  systemctl stop firewalld.service
```
3.确保节点是受信任的节点

### ubuntu heketi gluster 在创建 volume 时报错
报错信息:
```
Error: /usr/sbin/thin_check: execvp failed: No such file or directory
.......
Failed to activate thin pool .....
```
可能是缺少包 thin-provisioning-tools
```
  apt-get -y install thin-provisioning-tools
```
