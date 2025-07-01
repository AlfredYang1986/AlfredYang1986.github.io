# Deployments & StatefulSets 

# 1. 使用场景

- Deployment：
    - 适用于无状态应用（Stateless Applications）。
    - Pod 之间没有依赖关系，可以随意创建、删除或替换。
    - 典型的例子：Web 服务器、API 服务等。

- StatefulSet：
    - 适用于有状态应用（Stateful Applications）。
    - Pod 之间有明确的顺序和依赖关系，每个 Pod 有唯一的标识和稳定的网络标识。
    - 典型的例子：数据库（如 MySQL、PostgreSQL）、分布式存储系统（如 Zookeeper、Kafka）等。

# 2. 应用Pod上的问题

- Deployment：
    - Pod 的名称是随机的（如 web-app-59d8c5f6c4-abcde）。
    - Pod 之间没有固定的网络标识，IP 地址可能会变化。
    - Service 通常通过负载均衡将流量分发到任意 Pod。

- StatefulSet：
    - Pod 的名称是固定的、有序的（如 db-0、db-1、db-2）。
    - 每个 Pod 有稳定的网络标识（通过 Headless Service 实现），Pod 的 DNS 名称格式为 <pod-name>.<service-name>.<namespace>.svc.cluster.local。
    - Pod 的启动、更新和删除是有序的（按顺序进行）。

# 3. 存储应用上

- Deployment：
    - 通常使用共享存储（如 NFS、GlusterFS）或不使用持久化存储。
    - 如果使用 PersistentVolume（PV），所有 Pod 共享相同的存储卷。

- StatefulSet：
    - 每个 Pod 有独立的持久化存储（通过 PersistentVolumeClaim 模板实现）。
    - 存储与 Pod 的生命周期绑定，即使 Pod 被删除，存储仍然保留。
    - 适用于需要独立存储的场景（如数据库的每个实例需要独立的存储）。

# 4. 更新策略

- Deployment：
    - 支持滚动更新（Rolling Update），可以逐步替换旧的 Pod。
    - 更新时，Pod 是无序创建的，旧的 Pod 会被新的 Pod 替换。

- StatefulSet：
    - 支持有序更新（Ordered Update），Pod 按顺序逐个更新。
    - 更新时，Pod 会按照从高到低的顺序逐个替换（如 db-2 -> db-1 -> db-0）。
    - 也支持分区更新（Partitioned Update），可以只更新部分 Pod。

# 5. AutoScaling

- Deployment：
    - 扩缩容时，Pod 是无序创建或删除的。
    - 可以通过 kubectl scale 命令快速调整副本数。

- StatefulSet：
    - 扩缩容时，Pod 是按顺序创建或删除的。
    - 扩容时，Pod 会按顺序创建（如 db-0 -> db-1 -> db-2）。
    - 缩容时，Pod 会按逆序删除（如 db-2 -> db-1 -> db-0）。

# 6. 删除行为

- Deployment：
    - 删除 Deployment 时，所有关联的 Pod 也会被删除。
    - Pod 是无状态且可替换的，删除后可以重新创建。

- StatefulSet：
    - 删除 StatefulSet 时，默认不会删除关联的 Pod 和存储卷。
    - 需要手动删除 Pod 和 PersistentVolumeClaim（PVC），以确保数据安全。

# 7. 实践：部署数据库

```Yaml mongodb-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: acit-mongodb-demo
  name: acit-mongodb-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: acit-mongodb-demo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: acit-mongodb-demo
    spec:
      containers:
      - image: mongo
        name: acit-mongodb-demo
        args: ["--dbpath","/data/db"]
        livenessProbe:
          exec:
            command:
              - mongo
              - --disableImplicitSessions
              - --eval
              - "db.adminCommand('ping')"
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 6
        readinessProbe:
          exec:
            command:
              - mongo
              - --disableImplicitSessions
              - --eval
              - "db.adminCommand('ping')"
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 6
        volumeMounts:
        - name: "mongo-data-dir"
          mountPath: "/data/db"
      volumes:
      - name: "mongo-data-dir"
        persistentVolumeClaim:
          claimName: "mongodb-data-pvc"
```

# 8. 链接验证

使用 Mongodb Campass 验证

```Shell
kubectl get all -n k8s-acit-demo

kubectl port-forward pod/acit-mongodb-demo-659d9c8bcd-j6bl6 27017:27017 -n k8s-acit-demo
```

# 9. 作业

> 自学探针、选择器、secret等概念