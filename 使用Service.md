# 06 使用Service

## 01. 使用Service暴露数据库

```Yaml
---
apiVersion: v1
kind: Service
metadata:
  namespace: default
  labels: &ref_0
    app: mongodb
  name: mongodb-service
spec:
  type: ClusterIP
  sessionAffinity: None
  selector: *ref_0
  ports:
    - name: http-mongo
      protocol: TCP
      port: 27017
      targetPort: 27017
```

```Shell
kubectl apply -f mongodb-service
```

## 02. 自己的服务链接数据库

```Yaml

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: k8s-acit-demo
  labels:
    version: v1
    app: acit-jsonapi-demo-app
  name: acit-jsonapi-demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      version: v1
      app: acit-jsonapi-demo-app
  template:
    metadata:
      labels:
        version: v1
        app: acit-jsonapi-demo-app
    spec:
      containers:
      - name: acit-jsonapi-demo-app
        image: acit-jsonapi-demo:using
        imagePullPolicy: IfNotPresent
        ports:
        - name: http-app
          protocol: TCP
          containerPort: 8080
        env:
        - name: MONGO_HOST
          value: mongodb-service.default
        - name: MONGO_COLLECTION
          value: acit
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
```

```Shell

kubectl apply -f acit-jsonapi-demo.yaml
```

## 03. Service 暴露自己服务

```Yaml

---
apiVersion: v1
kind: Service
metadata:
  namespace: k8s-acit-demo
  labels:
    app: acit-jsonapi-demo-service
  name: acit-jsonapi-demo-service
spec:
  type: NodePort
  sessionAffinity: None
  selector:
    app: acit-jsonapi-demo-app
  ports:
    - name: http-jsonapi
      protocol: TCP
      port: 8080
      targetPort: 8080
```

```Shell

kubectl apply -f acit-jsonapi-service.yaml
```

## 04. 验证

Postman

