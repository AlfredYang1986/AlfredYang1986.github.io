# 08 Serving 服务 & Autoscaler

## 1. 拉模版代码

```Shell

git clone https://github.com/vladovel/knative-docs.git

cd knative-docs

cp docs/serving/samples/hello-world/helloworld-java-spring ../
```

## 2. Java 代码格式 【未来肯定会有跨语言调用】

```Shell
.mvn
  |-- maven-wrapper.properties          # maven 的配置
src
  |-- main                              # 代码目录
  |-- test                              # 单元测试目录
.dockerignore                           # docker 打包时需忽略的文件
.gitignore                              # git 代码提交时需忽略的文件
Dockerfile                              # 自动化打包文件
mvnw                                    # 本地调试启动代码
mvnw.cmd                                # 同上
pom.xml                                 # java包管理工具
README.md                               # 帮助文档
service.xml                             # 个人调试部署文档
```

## 3. 随便修一个代码

```Java

 @RestController
  class HelloworldController {
    @GetMapping("/")
    String hello() {
      return "Hello " + target + "!";
    }
  }
```

## 4. 修改个人调试部署文档

```Yaml

apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: helloworld-java-spring
  namespace: k8s-aict-demo
spec:
  template:
    spec:
      containers:
      - image: docker.io/alfredyang1986/helloworld-java-spring:0.0.1
        imagePullPolicy: IfNotPresent
        env:
        - name: TARGET
          value: "Spring Boot Sample v1"

```

>
> - Image 需要链接有效的制品库，当前使用docker hub
> - 未来会使用自己部署的

## 5. 编译

```Shell

docker build -t alfredyang1986/helloworld-java-spring:0.0.1 .
```

> 和Docker的编译没有区别

## 6. 本地调试

```Shell

./mvnw <参数>
```

> 依赖本地的环境
> 不要过度相信本地的调试，因为不同的环境能带来不同的效果
> 本地调试完，需要线上部署调试

## 7. 线上调试

```Shell

kubectl apply -f service.yaml -n k8s-aict-demo

kubectl get ksvc -n k8s-aict-demo. # 验证
```

```Shell

NAME                     URL                                                              LATESTCREATED                  LATESTREADY                    READY   REASON
helloworld-java-spring   http://helloworld-java-spring.k8s-aict-demo.127.0.0.1.sslip.io   helloworld-java-spring-00001   helloworld-java-spring-00001   True 
```

> 可以发现 虚拟DNS地址 “http://helloworld-java-spring.k8s-aict-demo.127.0.0.1.sslip.io”
> 直接访问，则可以访问服务

```Shell

curl http://helloworld-java-spring.k8s-aict-demo.127.0.0.1.sslip.io
```

## 8. 自动伸缩方式

### 两种自动伸缩方式：

1. Knative Pod Autoscaler (KPA)
    - Part of the Knative Serving core and enabled by default once Knative Serving is installed.
    - Supports scale to zero functionality.
    - Does not support CPU-based autoscaling.

2. Horizontal Pod Autoscaler (HPA)
    - Not part of the Knative Serving core, and you must install Knative Serving first.
    - Does not support scale to zero functionality.
    - Supports CPU-based autoscaling.

我们程序用量很小，而且需要 scale to zero ，也就当不需要的服务时，不保留服务实力，用于节约资源。所以我们会使用KPA。

### 配置

```Yaml

spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/class: "kpa.autoscaling.knative.dev"
        autoscaling.knative.dev/metric: "concurrency"
        autoscaling.knative.dev/target: "5"
        autoscaling.knative.dev/initialScale: "0"
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "5"
```

## 压力测试Demo

```Shell

hey -z 30s -c 50 \
  "http://helloworld-java-spring.k8s-aict-demo.127.0.0.1.sslip.io" \
  && kubectl get pods
```