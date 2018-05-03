#### 关闭虚拟内存
``` swapoff -a ```  
永久关闭：   
在 linux 文件中：``` /etc/fstab ```  
注掉该行 ：``` /dev/mapper/centos-swap swap                    swap    defaults        0 0```  
#### 关闭 seLinux
``` setenforce 0 ```

***懒得写了  未完待续 ....***
