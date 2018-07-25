待更
```
To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace chine mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=$(kubectl get nodes --namespace chine -o jsonpath='{.items[0].status.addresses[0].address}')
    MYSQL_PORT=$(kubectl get svc --namespace chine mysql -o jsonpath='{.spec.ports[0].nodePort}')

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
d
```
