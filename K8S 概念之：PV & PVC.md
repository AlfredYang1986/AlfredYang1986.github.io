# K8S 概念之：PV & PVC

# 1. 持久卷（PersistentVolume, PV)

PV是Kubernetes集群中的一块网络存储，它独立于Pod存在，可以被多个Pod共享或独占。PV可以被视为集群级别的资源，用于存储Pod产生的数据。PV可以是各种存储系统，如云提供商的存储、NFS、iSCSI、本地存储等。PV由管理员创建，并配置其细节，如容量、访问模式（ReadWriteOnce、ReadOnlyMany、ReadWriteMany）、存储类别等。

- 容量：指定PV的存储容量。
- 访问模式：指定PV的访问模式，ReadWriteOnce表示一次只能一个Pod写入，ReadOnlyMany表示多个Pod可以同时读取但不能写入，ReadWriteMany表示多个Pod可以同时读写。
- 存储类别：指定PV的存储类别，用于动态创建PV时选择存储后端。


# 2. 持久卷声明（PersistentVolumeClaim, PVC)

PVC是用户对持久存储的请求声明，它定义了Pod对存储的需求。PVC可以指定所需的存储容量、访问模式等参数，但通常不需要指定具体的PV，而是通过标签选择器来动态匹配PV。PVC的存在使得Pod与具体的存储实现解耦，提高了可移植性。

- 存储容量：指定PVC所需的存储容量。
- 访问模式：指定PVC的访问模式，与PV的访问模式相匹配。
- 存储类别：指定PVC所需的存储类别，用于动态创建PV时选择存储后端。
- 标签选择器：用于动态匹配PV的标签。

PVC与PV之间是一种声明与提供的关系。PVC声明了对存储资源的需求，而PV则是提供这些资源的实际载体。当PVC被创建时，Kubernetes会尝试将其与满足其要求的PV进行绑定。匹配的过程是根据PVC的标签选择器和PV的标签进行匹配，只有匹配成功的PV才能被绑定到PVC。一旦绑定成功，Pod可以通过PVC访问PV提供的存储资源。如果没有合适的PV可以绑定，PVC将处于Pending状态，直到有合适的PV可用为止。

# 3. PV => PVC 解决存算分离问题

PV和PVC的设计实现了Pod与存储资源的解耦，使得Pod可以独立于存储资源的变化而运行。这种设计提高了系统的灵活性和可移植性。

- 动态匹配与绑定：PVC声明了对持久卷的需求，而PV则提供了实际的存储资源。Kubernetes会自动将PVC与合适的PV进行匹配和绑定。这种动态匹配机制使得用户无需关心具体的PV细节，只需声明对存储资源的需求即可。
- 按需分配：通过PVC，可以实现存储资源的按需分配。用户可以根据应用的需求动态申请存储资源，而无需提前准备或分配存储资源。这种按需分配机制提高了资源利用率和系统的可扩展性。
- 生命周期管理：PV和PVC的生命周期管理由Kubernetes负责，包括资源的创建、绑定、使用和回收等阶段。这种生命周期管理机制确保了存储资源的有效使用和回收。

# 4. 实践：部署存储

mongodb-pv.yaml
```Yaml 
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-data-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /Users/alfredyang/Desktop/data
```

```Shell
kubectl apply -f mongoddb-pv.yaml
```

mongodb-pvc.yaml
```Yaml 
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-data-pvc 
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce 
  volumeName: pv
  resources:
    requests:
      storage: 1Gi
```

```Shell
kubectl apply -f mongoddb-pvc.yaml -n <namespace>
```
