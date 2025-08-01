---
title: 通过Ingress Annotation实现高阶流量治理
keywords: [Ingress Annotation]
description: 在Higress上使用Ingress并借助Annotation实现高阶流量治理。
---

本篇文档介绍如何在Higress上使用Ingress并借助Annotation实现高阶流量治理。

## 前提条件
- [安装Higress](./quickstart.md)
- 已拥有一个Kubernetes集群，且配置了kubectl命令行工具

## 灰度发布
Higress提供复杂的路由处理能力，支持基于Header、Cookie以及权重的灰度发布功能。灰度发布功能可以通过设置注解来实现，为了启用灰度发布功能，需要设置注解`higress.io/canary: "true"`。通过不同注解可以实现不同的灰度发布功能。
> 说明：当多种方式同时配置时，灰度方式选择优先级为：基于Header > 基于Cookie > 基于权重（从高到低）。

### 基于Header灰度发布
- 只配置`higress.io/canary-by-header`：基于Request Header的名称进行流量切分。当请求包含该Header并其值为always时，请求流量会被分配到灰度服务入口；其他情况时，请求流量不会分配到灰度服务。
- 同时配置`higress.io/canary-by-header`和`higress.io/canary-by-header-value`：基于Request Header的名称和值进行流量切分。当请求中的header的名称和header的值与该配置匹配时，请求流量会被分配到灰度服务；其他情况时，请求流量不会分配到灰度服务。
> Higress灰度发布时支持多个版本服务（无上限）。

1. 请求Header为`higress：always`时将访问灰度服务demo-service-canary；其他情况将访问正式服务demo-service，配置如下：
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/canary: "true"
    higress.io/canary-by-header: "higress"
  name: demo-canary
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service-canary
                port: 
                  number: 80
            path: /hello
            pathType: Exact
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /hello
            pathType: Exact   
```

2. 请求Header为`higress：v1`时将访问灰度服务demo-service-canary-v1；请求Header为`higress：v2`时将访问灰度服务demo-service-canary-v2；其他情况将访问正式服务demo-service，配置如下：
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/canary: "true"
    higress.io/canary-by-header: "higress"
    higress.io/canary-by-header-value: "v1"
  name: demo-canary-v1
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service-canary-v1
                port: 
                  number: 80
            path: /hello
            pathType: Exact
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/canary: "true"
    higress.io/canary-by-header: "higress"
    higress.io/canary-by-header-value: "v2"
  name: demo-canary-v2
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service-canary-v2
                port: 
                  number: 80
            path: /hello
            pathType: Exact
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /hello
            pathType: Exact
```

### 基于Cookie灰度发布
- higress.io/canary-by-cookie：基于Cookie的Key进行流量切分。当请求的Cookie中含有该Key且其值为always时，请求流量将被分配到灰度服务；其他情况时，请求流量将不会分配到灰度服务。
> 说明：基于Cookie的灰度发布不支持自定义设置Key对应的值，只能是always。

请求的Cookie为`demo=always`时将访问灰度服务demo-service-canary；其他情况将访问正式服务demo-service。配置如下：
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/canary: "true"
    higress.io/canary-by-cookie: "demo"
  name: demo-canary
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service-canary
                port: 
                  number: 80
            path: /hello
            pathType: Exact
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /hello
            pathType: Exact
```

### 基于权重灰度发布
- higress.io/canary-weight：设置请求到指定服务的百分比（值为0~100的整数）
- higress.io/canary-weight-total：设置权重总和，默认为100

配置灰度服务demo-service-canary-v1的权重为30%，配置灰度服务demo-service-canary-v2的权重为20%，配置正式服务demo-service的权重为50%。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/canary: "true"
    higress.io/canary-weight: "30"
  name: demo-canary-v1
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service-canary-v1
                port: 
                  number: 80
            path: /hello
            pathType: Exact
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/canary: "true"
    higress.io/canary-weight: "20"
  name: demo-canary-v2
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service-canary-v2
                port: 
                  number: 80
            path: /hello
            pathType: Exact
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /hello
            pathType: Exact
```

## 跨域
跨域资源共享CORS（Cross-Origin Resource Sharing）是指允许Web应用服务器进行跨域访问控制，从而实现跨域数据安全传输。
- higress.io/enable-cors："true" or "false"。开启或关闭跨域。
- higress.io/cors-allow-origin：允许的第三方站点，支持泛域名，逗号分隔；支持通配符*。默认值为*，即允许所有第三方站点。
- higress.io/cors-allow-methods：允许的请求方法，如GET、POST，逗号分隔；支持通配符*。默认值为GET, PUT, POST, DELETE, PATCH, OPTIONS。
- higress.io/cors-allow-headers：允许的请求头部，逗号分隔；支持通配符*。默认值为DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization。
- higress.io/cors-expose-headers：允许的响应头部，逗号分隔。
- higress.io/cors-allow-credentials："true" or "false"。是否允许携带凭证信息。默认允许。
- higress.io/cors-max-age：预检结果的最大缓存时间，单位为秒；默认值为1728000。


跨域请求被限制为只能来自example.com域的请求，并且HTTP方法只能是GET和POST，允许的请求头部为X-Foo-Bar，不允许携带凭证信息。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/enable-cors: "true"
    higress.io/cors-allow-origin: "example.com"
    higress.io/cors-allow-methods: "GET,POST"
    higress.io/cors-allow-headers: "X-Foo-Bar"
    higress.io/cors-allow-credentials: "false"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /hello
            pathType: Exact
```

## Rewrite重写Path和Host
在请求转发给目标后端服务之前，重写可以修改原始请求的路径（Path）和主机域（Host)。
- higress.io/rewrite-target：重写Path，支持捕获组（Capture Group)。
- higress.io/upstream-vhost：重写Host。

### Rewrite重写Path
1. 将请求`example.com/test`在转发至后端服务之前，重写为`example.com/dev`
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/rewrite-target: "/dev"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

2. 将请求`example.com/v1/app`在转发至后端服务之前，去掉Path前缀/v1
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/rewrite-target: "/$2"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /v1(/|$)(.*)
            pathType: ImplementationSpecific
```

3. 将请求`example.com/v1/app`在转发至后端服务之前，将Path前缀/v1更改为/v2
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/rewrite-target: "/v2/$2"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /v1(/|$)(.*)
            pathType: ImplementationSpecific
```

### Rewrite重写Host
将请求`example.com/test`在转发至后端服务之前，重写为`test.com/test`
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/upstream-vhost: "test.com"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## 重定向
通过重定向可以将原始客户端请求更改为目标请求。
### 配置HTTP重定向至HTTPS
- higress.io/ssl-redirect：HTTP重定向到HTTPS
- higress.io/force-ssl-redirect: HTTP重定向到HTTPS
> 说明：Higress对于以上两个注解不区分对待，都是强制将HTTP重定向到HTTPS。

将请求`http://example.com/test`重定向为`https://example.com/test`。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/ssl-redirect: "true"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

### 永久重定向
- higress.io/permanent-redirect：永久重定向的目标url，必须包含scheme（http or https)。
- higress.io/permanent-redirect-code：永久重定向的HTTP状态码，默认为301。

将请求`http://example.com/test`永久重定向为`http://example.com/app`。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/permanent-redirect: "http://example.com/app"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

### 临时重定向
- higress.io/temporal-redirect：临时重定向的目标url，必须包含scheme（http or https)。
  
将请求`http://example.com/test`临时重定向为`http://example.com/app`。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/temporal-redirect: "http://example.com/app"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## Header控制
通过Header控制，您可以在转发请求到后端服务之前对请求Header进行增删改，在收到响应转发给客户端时对响应Header进行增删改。
### 请求Header控制
- higress.io/request-header-control-add：请求在转发给后端服务时，添加指定Header。若该Header存在，则其值拼接在原有值后面。语法如下：
  - 单个Header：Key Value
  - 多个Header：使用yaml特殊符号 |，每对Key Value单独处于一行
- higress.io/request-header-control-update：请求在转发给后端服务时，修改指定Header。若该header存在，则其值覆盖原有值。语法如下：
  - 单个Header：Key Value
  - 多个Header：使用yaml特殊符号 |，每对Key Value单独处于一行
- higress.io/request-header-control-remove：请求在转发给后端服务时，删除指定Header。语法如下：
  - 单个Header：Key
  - 多个Header：逗号分隔

1. 对于请求`example.com/test`添加两个Header，`foo: bar`和`test: true`。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/request-header-control-add: |
      foo bar
      test true
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

2. Header控制可以结合灰度发布，对灰度流量进行染色。请求Header为`higress：v1`时将访问灰度服务demo-service-canary-v1，并添加Header，`stage: gray`；其他情况将访问正式服务demo-service，并添加Header，`stage: production`。配置如下：
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/canary: "true"
    higress.io/canary-by-header: "higress"
    higress.io/canary-by-header-value: "v1"
    higress.io/request-header-control-add: "stage gray"
  name: demo-canary-v1
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service-canary-v1
                port: 
                  number: 80
            path: /hello
            pathType: Exact
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/request-header-control-add: "stage production"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /hello
            pathType: Exact
```

### 响应Header控制
- higress.io/response-header-control-add：请求在收到后端服务响应之后并且转发响应给客户端之前，添加指定Header。若该Header存在，则其值拼接在原有值后面。语法如下：
  - 单个Header：Key Value
  - 多个Header：使用yaml特殊符号 |，每对Key Value单独处于一行
- higress.io/response-header-control-update：请求在收到后端服务响应之后并且转发响应给客户端之前，修改指定Header。若该header存在，则其值覆盖原有值。语法如下：
  - 单个Header：Key Value
  - 多个Header：使用yaml特殊符号 |，每对Key Value单独处于一行
- higress.io/response-header-control-remove：请求在收到后端服务响应之后并且转发响应给客户端之前，删除指定Header。语法如下：
  - 单个Header：Key
  - 多个Header：逗号分隔

对于请求`example.com/test`的响应删除`Header：req-cost-time`。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/response-header-control-remove: "req-cost-time"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## 重试
Higress提供路由级别的重试设置，可以为出错的请求调用自动进行重试。您可以按需设置重试条件，例如建立连接失败，或者后端服务不可用以及对指定HTTP状态码的响应等进行请求重试。
- higress.io/proxy-next-upstream-tries：请求的最大重试次数。默认3次。
- higress.io/proxy-next-upstream-timeout：请求重试的超时时间，单位秒。默认未配置超时时间。
- higress.io/proxy-next-upstream：请求重试条件，逗号分隔；默认值为"error,timeout"。合法值如下：
  - error：建立连接失败，请求出错5xx。
  - timeout：建立连接超时，请求出错5xx。
  - invalid_header：请求出错5xx。
  - http_xxx：针对具体响应状态码的情况进行重试。例如http_502，http_403。
  - non_idempotent：对于非幂等请求出错时进行重试。默认情况下，Higress针对非幂等POST、PATCH请求出错时不会进行重试，配置non_idempotent可以开启重试。
  - off：关闭重试。

设置`example/test`请求的最大重试次数为2，重试超时时间为5s，只有在响应状态码为502才重试，并且开启非幂等重试。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/proxy-next-upstream-tries: "2"
    higress.io/proxy-next-upstream-timeout: "5"
    higress.io/proxy-next-upstream: "http_502,non_idempotent"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## 超时
Higress提供路由级别的超时设置，与nginx ingress不同，没有区分连接/读写超时，而是面向的接口处理总延时进行配置，在未进行配置时默认不限制，例如后端未返回应答，网关将无限等待。

设置超时时间为5s：

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/timeout: "5"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## 单机限流

> **提示**
> Higress商业版具备全局限流能力，详情查看[商业版文档](https://help.aliyun.com/zh/mse/user-guide/advanced-usage-of-mse-ingress?spm=a2c4g.11186623.0.0.2e3a3db3eYcspD#862f08d03d4d3)中全局限流一节的介绍

支持针对路由级别的单机限流策略，在设定的时间周期内，限制每个网关副本匹配在某个路由上的请求数量不大于阈值。该限流是针对单机级别，即配置的阈值在每个网关实例进行流控。

- higress.io/route-limit-rpm：该Ingress定义的路由在每个网关实例上每分钟最大请求次数。瞬时最大请求次数为该值乘以limit-burst-multiplier。
- higress.io/route-limit-rps：该Ingress定义的路由在每个网关实例上每秒最大请求次数。瞬时最大请求次数为该值乘以limit-burst-multiplier。
- higress.io/route-limit-burst-multiplier：瞬时最大请求次数的因子，默认为5。

例如：

限制example.com/test的请求每分钟最大请求数为100，瞬时请求数200。配置如下：
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/route-limit-rpm: "100"
    higress.io/route-limit-burst-multiplier: "2"
  name: demo
spec:
  ingressClassName: mse
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

限制example.com/test的请求每秒最大请求数为10，瞬时请求数50。配置如下：
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/route-limit-rps: "10"
    # 默认为5
    # higress.io/route-limit-burst-multiplier: "5"
  name: demo
spec:
  ingressClassName: mse
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```


## 配置后端服务协议：HTTPS或GRPC
Higress默认使用HTTP协议转发请求到后端业务容器。当您的业务容器为HTTPS协议时，可以通过使用注解`higress.io/backend-protocol: "HTTPS"`；当您的业务容器为GRPC服务时，可以通过使用注解`higress.io/backend-protocol: "GRPC"`。
> 说明：相比Nginx Ingress的优势，如果您的后端服务所属的K8s Service资源中关于Port Name的定义为grpc或http2，您无需配置注解higress.io/backend-protocol: "GRPC"，Higress会自动使用GRPC或者HTTP2。

1. 请求`example.com/test`转发至后端服务使用HTTPS协议。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/backend-protocol: "HTTPS"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /
            pathType: Exact
```

2. 请求`example/grpcbin.GRPCBin`转发至后端服务使用GRPC协议。

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/backend-protocol: "GRPC"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /grpcbin.GRPCBin
            pathType: Prefix
```

## 配置后端服务的负载均衡算法
负载均衡决定着网关在转发请求至后端服务时如何选择节点。
### 普通负载均衡算法
- higress.io/load-balance：后端服务的普通负载均衡算法。默认为round_robin。合法值如下：
  - round_robin：基于轮询的负载均衡。
  - least_conn：基于最小请求数的负载均衡。
  - random：基于随机的负载均衡。

设置后端服务demo-service的负载均衡算法为least_conn。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/load-balance: "least_conn"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /order
            pathType: Exact
```

### 基于一致性Hash的负载均衡算法
基于一致性Hash的负载均衡算法具备请求亲和性，具有相同特征的请求会始终负载到相同节点上。Higress支持基于部分Nginx 变量、请求Header和请求路径参数作为Hash Key。
- higress.io/upstream-hash-by：基于一致Hash的负载均衡算法。Higress支持以下几种形式：
  - 支持配置部分nginx变量：
    - $request_uri：请求的Path（包括路径参数）作为Hash Key
    - $host：请求的Host作为Hash Key
    - $remote_addr：请求的客户端IP作为Hash Key
  - 基于请求header的一致性Hash。您只需配置为$http_headerName。
  - 基于请求路径参数的一致性Hash。您只需配置为$arg_varName。

1. 基于请求的客户端IP作为Hash Key，同一个客户端IP的请求始终负载到同一个节点。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/upstream-hash-by: "$remote_addr"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

2. 基于请求Header x-stage作为Hash key，带有x-stage头部的请求且值相同的请求始终负载到同一个节点。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/upstream-hash-by: "$http_x-stage"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

3. 基于请求路径参数 x-stage作为Hash key，带有路径参数x-stage的请求且值相同的请求始终负载到同一个节点。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/upstream-hash-by: "$arg_x-stage"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## Cookie亲和性 （会话保持）
具备相同的Cookie的请求会被网关始终负载到同一个节点，并且如果第一次访问携带Cookie，Higress会在第一次响应时为客户端生成一个Cookie，用来保证后续的请求被网关始终负载到相同节点。
- higress.io/affinity：亲和性种类，目前只支持cookie，默认为cookie。
- higress.io/affinity-mode：亲和性模式，Higress目前只支持balanced模式，默认为balanced模式。
- higress.io/session-cookie-name：配置指定Cookie的值作为Hash Key，默认为INGRESSCOOKIE
- higress.io/session-cookie-path：当指定Cookie不存在，生成的Cookie的Path值，默认为/
- higress.io/session-cookie-max-age：当指定Cookie不存在，生成的Cookie的过期时间，单位为秒，默认为Session会话级别。
- higress.io/session-cookie-expires：当指定Cookie不存在，生成的Cookie的过期时间，单位为秒，默认为Session会话级别。
> 说明：max-age和expires都可以用来指定cookie过期时间。当session-cookie-max-age和session-cookie-expires同时配置时，Higress优先选取session-cookie-max-age作为过期时间。

1. 开启Cookie亲和性，利用Higress的默认配置，即Cookie的名字为INGRESSCOOKIE，Path为/，Cookie的生命周期为Session会话级别。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/affinity: "cookie"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

2. 开启Cookie亲和性，Cookie的名字为test，Path为/，Cookie的过期时间为10s。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/affinity: "cookie"
    higress.io/session-cookie-name: "test"
    higress.io/session-cookie-max-age: "10"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## 网关与客户端双向认证(MTLS)

当开启HTTPS时，网关与客户端默认是基于单向TLS认证，可以通过下面注解开启双向TLS认证，即让服务端同时校验客户端的合法性。

- higress.io/auth-tls-secret: 网关使用的CA证书，用于验证MTLS握手期间，客户端提供的证书。该注解主要应用于网关需要验证客户端身份的场景。secret名字格式必须是：(该域名证书所在的secret的名字)-cacert

例如:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    # 这里的要求是必须以域名上配置证书secret名称后缀加上-cacert
    higress.io/auth-tls-secret: tls-secret-cacert
  name: bar
  namespace: default
spec:
  ingressClassName: higress
  rules:
  - host: bar.com
    http:
      paths:
      - backend:
          service:
            name: bar-service
            port:
              number: 5678
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - bar.com
    secretName: tls-secret
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: foo
  namespace: default
spec:
  ingressClassName: higress
  rules:
  - host: foo.com
    http:
      paths:
      - backend:
          service:
            name: foo-service
            port:
              number: 5678
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - foo.com
    secretName: tls-secret
```

tls-secret-cacert 这个 secret 的内容中必须有 cacert 这个key，例如：
```yaml
apiVersion: v1
data:
  cacert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURDekNDQWZPZ0F3SUJBZ0lVQ0tMSGM5SFhBbEFYNUdFR2dHVk1zVXhzUFhBd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0ZURVRNQkVHQTFVRUF3d0tSWGhoYlhCc1pTQkRRVEFlRncweU16RXlNakl3T1RNek1ESmFGdzB5TkRFeQpNakV3T1RNek1ESmFNQlV4RXpBUkJnTlZCQU1NQ2tWNFlXMXdiR1VnUTBFd2dnRWlNQTBHQ1NxR1NJYjNEUUVCCkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDaW1jaWQ4VWx4VzA4a1RTcmc1UnAzTlMvSmFMQWt3bVZzeWdEanc0TUEKSjh6Q2FWWHFmU2xpbCtTRFdLcllRYUtPQ1JRWjlWdXBwODl4UnRJTkpUVmlBZUpHYzh6SDY2Sy82aUZJZ2N4ZQplczVaaDdqQXdENzZ4eEtMUjJPbkRSb0xpVlFVOGxkekVNclVHRytCOXJ1TzFsNjkxNlRjQ0dqS3VGUklQNzJCCjlJcEI0ekxZUUNLWldmZ1cxVmU0alpYTUZ0MVhUc0dWdkhCaEt0MSt5eXMrNnc3WndxMW43NysxdXcya2dmM3cKaXNCbXBzTlRPVVJSZzVvdEdYVUUxaGl3dC9KeW9PQkt1YmVyY2dwd05OYzAvNHZ6eWRHMm83UFFpTHcyallPbwppbFptYUZzVXEyclU4S0hCdWlSbVkyTXlOWEU2R0liY29sVGZRQWM2NE5EWEFnTUJBQUdqVXpCUk1CMEdBMVVkCkRnUVdCQlNOZC9vYTkyemFGWFNaRVJoRXJMSElqRE8zYWpBZkJnTlZIU01FR0RBV2dCU05kL29hOTJ6YUZYU1oKRVJoRXJMSElqRE8zYWpBUEJnTlZIUk1CQWY4RUJUQURBUUgvTUEwR0NTcUdTSWIzRFFFQkN3VUFBNElCQVFBTwpjMDNIYWFDRHhhR0phdzlrYkJxMW1DbUVvR3pWZ0loYkhveTQ0Q0IxbGpnS0xOWHo2bnZ5bDVCdWRzWXJkT1lXCmJMMEJGdGxLbWRqeUFHemtnOThGSkpVNExTVFM1ZDNySlBkQU1lcXFOQ2R5NVh0c2l2VDlIbzh5QVBiUGFmZlkKTmozN29JVEQrdXhQbTNVMGhOTU5YSGdGdnJ4bGV6U2MyOHFWL1VxVDBWcnNJR3IyblNiaEYrR3g1WS92aTZocQp5RTJsYWJXdDQ3VlBYcTNFL2lHRWhTSmFndTdhN2xBSDhYWWlqMUtCMkU4bjlERy80R2ZDMEVpTnNXbUpzWVNjCk9tdXlmRGpXaHQ2LzlUVkh4YkNZMzZGQ08vOTcraThqUGhxVlkxRlJzUG5WRUtiRXBNemdXb3Y0UXNKeHoxS3gKdHN2eHlVRnJsaU5wUk1OQmVEODIKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
kind: Secret
metadata:
  name: tls-secret-cacert
  namespace: default
type: Opaque
```

## 网关与后端服务双向认证（MTLS)
默认情况下，Higress默认使用HTTP协议转发请求到后端业务容器。你可以通过使用注解`higress.io/backend-protocol: "HTTPS"`配置Higress访问后端服务使用HTTPS协议，但这是单向TLS，只有后端服务提供的证书。另一种更安全的模式是零信任，网关会验证后端服务的证书是否合法，同样后端服务也会验证网关提供的证书是否合法，这就是MTLS，网关与后端服务进行双向认证。

- higress.io/proxy-ssl-secret：网关使用的客户端证书，用于后端服务对网关进行身份认证，格式为 secretNamespace/secretName。
- higress.io/proxy-ssl-name：TLS握手期间使用的SNI。
- higress.io/proxy-ssl-server-name：on or off。开启或关闭TLS握手期间使用SNI。

网关与后端服务进行双向认证，网关使用的secret name为gateway-cert，命名空间为default。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/proxy-ssl-secret: "default/gateway-cert"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## 配置客户端到网关之间的TLS版本以及加密套件

目前，Higress 默认最小TLS版本为TLSv1.0，默认最大TLS版本为TLSv1.3，默认加密套件为：

ECDHE-ECDSA-AES128-GCM-SHA256

ECDHE-RSA-AES128-GCM-SHA256

ECDHE-ECDSA-AES128-SHA

ECDHE-RSA-AES128-SHA

AES128-GCM-SHA256

AES128-SHA

ECDHE-ECDSA-AES256-GCM-SHA384

ECDHE-RSA-AES256-GCM-SHA384

ECDHE-ECDSA-AES256-SHA

ECDHE-RSA-AES256-SHA

AES256-GCM-SHA384

AES256-SHA

您可以通过以下注解为特定的域名设置最小或者最大TLS版本以及加密套件。

- higress.io/tls-min-protocol-version: 指定TLS的最小版本，默认值为TLSv1.0。合法值如：TLSv1.0/TLSv1.1/TLSv1.2/TLSv1.3
- higress.io/tls-max-protocol-version: 指定TLS的最大版本，默认值为TLSv1.3。合法值如：TLSv1.0/TLSv1.1/TLSv1.2/TLSv1.3
- higress.io/ssl-cipher: 指定TLS的加密套件，可以指定多个英文冒号分隔，仅当TLS握手时采用TLSv1.0~1.2生效

例如，对于域名example.com，设置TLS最小版本为TLSv1.2，最大版本为TLSv1.2。配置如下：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/tls-min-protocol-version: "TLSv1.2"
    higress.io/tls-max-protocol-version: "TLSv1.2"
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

## 流量镜像

通过配置流量镜像，可以复制流量到指定服务，常用于操作审计和流量测试等场景。

- higress.io/mirror-target-service：复制流量转发到指定镜像服务。服务格式为：namespace/name:port。
  - namespace: K8s Service所在的命名空间，可选，默认为Ingress所在的命名空间。
  - name：K8s Service的名称，必选。
  - port：待转发至K8s Service的端口，可选，默认为第一个端口。
- higress.io/mirror-percentage：复制流量的比例。可配置的值的范围为：0~100，默认100。

> **重要提醒： 复制的流量在转发给目标服务时，原始请求中的Host会被自动加上-shadow后缀。**

下面例子是将example.com/test的流量复制并转发到目标服务：命名空间为test，服务名为app，端口为8080，且复制比例为100%。
本示例中，复制的流量在转发给目标服务时，Host会被自动改写为example.com-shadow：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/mirror-target-service: test/app:8080
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```

下面例子是将example.com/test的流量复制并转发到目标服务：命名空间为test，服务名为app，端口为8080，且复制比例为10%。
本示例中，复制的流量在转发给目标服务时，Host会被自动改写为example.com-shadow：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    higress.io/mirror-target-service: test/app:8080
    higress.io/mirror-percentage: 10
  name: demo
spec:
  ingressClassName: higress
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: demo-service
                port: 
                  number: 80
            path: /test
            pathType: Exact
```


