# 05 Dockerfile 与 自建服务的发布

## 1. 构建自己的服务镜像

这里我使用JsonAPI来做Demo。JSON:API是一个结构化API，其目的是达成 Object2Object 的前后端编程模式，即快速方便的完成所有CURD操作。

JSON:API是一种用于构建和设计RESTful API的规范，它定义了一套约定和规则，用于描述API的数据格式、关联关系、资源的获取和修改等操作。JSON:API旨在提供一种一致性和标准化的方式来设计和开发API，以提高API的可读性、可维护性和可扩展性。

JSON:API的优势包括：

- 一致性：JSON:API提供了一套统一的规范和约定，使得不同API之间的数据格式和操作方式保持一致。
- 关联关系：JSON:API支持定义资源之间的关联关系，使得客户端可以方便地获取和操作相关资源。
- 性能优化：JSON:API提供了一些性能优化的机制，如批量操作和数据缓存，可以提高API的性能和响应速度。

JSON:API的应用场景包括：

- 多资源获取：通过JSON:API的关联关系，可以方便地获取和展示多个相关资源的数据，减少了多次请求的开销。
- 数据修改：JSON:API提供了一套标准的方式来修改资源的数据，包括创建、更新和删除操作，使得API的数据修改更加一致和可控。
- 客户端开发：JSON:API的一致性和规范性使得客户端开发更加简单和高效，可以减少重复的代码和逻辑。

```Dockfile

FROM node:18.16.1
RUN npm config set registry https://registry.npmmirror.com
WORKDIR /app
COPY ./ .
RUN npm install
EXPOSE 8080
ENTRYPOINT [ "npm", "run", "start" ]
```

```Shell

docker build -t acit-jsonapi-demo:using .

docker image ls
```

## 02. 部署自己的服务

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
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
```

```Shell

kubectl apply -f acit-jsonapi-demo-deployment.yaml
```