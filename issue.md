### x509: certificate signed by unknown authority
可能由于jenkins 升级证书问题  
way: 查看 jenkins master节点,一般在 /root/.kube/下面,文件为 config,将此文件(即证书配置)拷到要用的机器上
