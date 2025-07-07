# 10 Eventing

## 1. init kafka 

```Shell

git clone https://github.com/ramhiser/kafka-kubernetes.git

kubectl create -f namespace-kafka.yaml
kubectl config set-context kafka --namespace=kafka-cluster --cluster=${CLUSTER_NAME} --user=${USER_NAME}
kubectl config use-context kafka
```

### 验证

```Shell

# 安装 kafka 客户端命令行
brew install kcat
```

## 2. install eventing

```Shell

# crds
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.18.2/eventing-crds.yaml

# core
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.18.2/eventing-core.yaml

# Kafka controller
kubectl apply --filename https://github.com/knative-extensions/eventing-kafka-broker/releases/download/knative-v1.18.0/eventing-kafka-controller.yaml

# Kafka Broker data plane
kubectl apply --filename https://github.com/knative-extensions/eventing-kafka-broker/releases/download/knative-v1.18.0/eventing-kafka-broker.yaml

kubectl get deployments.apps -n knative-eventing
```

## 3. install Kafka Broker

```Shell

# create broker 
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  annotations:
    # case-sensitive
    eventing.knative.dev/broker.class: Kafka
    # Optional annotation to point to an externally managed kafka topic:
    # kafka.eventing.knative.dev/external.topic: alfred-demo
  name: eventing-demo
  namespace: knative-eventing
spec:
  # Configuration specific to this broker.
  config:
    apiVersion: v1
    kind: ConfigMap
    name: kafka-broker-config
    namespace: knative-eventing
  # Optional dead letter sink, you can specify either:
  #  - deadLetterSink.ref, which is a reference to a Callable
  #  - deadLetterSink.uri, which is an absolute URI to a Callable (It can potentially be out of the Kubernetes cluster)
  # delivery:
  #   deadLetterSink:
  #     ref:
  #       apiVersion: serving.knative.dev/v1
  #       kind: Service
  #       name: dlq-service

```

```Shell

# create cm
apiVersion: v1
kind: ConfigMap
metadata:
  name: kafka-broker-config
  namespace: knative-eventing
data:
  # Number of topic partitions
  default.topic.partitions: "1"
  # Replication factor of topic messages.
  default.topic.replication.factor: "1"
  # A comma separated list of bootstrap servers. (It can be in or out the k8s cluster)
  bootstrap.servers: "kafka-service.knative-eventing.svc.cluster.local:9092"
  # Here is our retention.ms config
  default.topic.config.retention.ms: "3600"
```

> 因为实现原因，Kafka必须和Knative在同一个namespace下，切记！！！！

## 4. Create Trigger

```Yaml

apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: eventing-demo-trigger
  namespace: knative-eventing
spec:
  broker: eventing-demo
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: helloworld-java-spring
      namespace: k8s-aict-demo
```

## 5. 验证

```Yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: curl
  name: curl
  namespace: knative-eventing
spec:
  containers:
    - image: yurakolam/busyboxplus:curl
      imagePullPolicy: IfNotPresent
      name: curl
      resources: {}
      stdin: true
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      tty: true
```

```Shell

kubectl -n knative-eventing attach curl -it

curl -v "http://kafka-broker-ingress.knative-eventing.svc.cluster.local/knative-eventing/eventing-demo" \
  -X POST \
  -H "Ce-Id: say-hello" \
  -H "Ce-Specversion: 1.0" \
  -H "Ce-Type: greeting" \
  -H "Ce-Source: not-sendoff" \
  -H "Content-Type: application/json" \
  -d '{"msg":"Hello Knative!"}'
```

## 6. event sequence flow

![](https://knative.dev/docs/eventing/flows/sequence/sequence-with-broker-trigger/sequence-with-broker-trigger.png)

[当作作业吧](https://knative.dev/docs/eventing/flows/sequence/sequence-with-broker-trigger/#prerequisites)
