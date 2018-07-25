待更
```
Credentials:

    Username      : admin
    echo Password      : $(kubectl get secret --namespace chine rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)
    echo ErLang Cookie : $(kubectl get secret --namespace chine rabbitmq -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 --decode)

RabbitMQ can be accessed within the cluster on port 5672 at rabbitmq.chine.svc.cluster.local

To access for outside the cluster, perform the following steps:

Create a proxy server between localhost and the Kubernetes API Server:

    kubectl proxy --port 8002

To Access the RabbitMQ AMQP port:

    URL : amqp://127.0.0.1:8002/api/v1/proxy/namespaces/chine/services/rabbitmq:5672/

To Access the RabbitMQ Management interface:

    URL : http://127.0.0.1:8002/api/v1/proxy/namespaces/chine/services/rabbitmq:15672/

```
