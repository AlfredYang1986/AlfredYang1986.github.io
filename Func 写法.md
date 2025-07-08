# 11 Func 写法

## 安装 knative plugin func

```Shell

brew tap knative-extensions/kn-plugins

brew install func
```

## 创建 Func

```Shell

func create -l <language> <function-name>

func create -l springboot demo-func
```

> The coordinates for the image registry can be configured through an environment variable (FUNC_REGISTRY) as well.

## 调用 Func

```Shell

kn func run --build=false

func deploy --build=false
```

