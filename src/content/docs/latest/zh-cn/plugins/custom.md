---
title: 自定义插件
keywords: [higress,custom,plugin]
description: 自定义插件配置参考
---

## 编辑 Wasm 文件

1. 用于使用 Go 语言开发 Higress 的 Wasm 插件的[SDK](https://github.com/alibaba/higress/tree/main/plugins/wasm-go)
2. [插件开发示例](../user/wasm-go.md)

## 构建 Wasm 镜像

你也可以选择先在本地将 wasm 构建出来，再拷贝到 Docker 镜像中。这要求你要先在本地搭建构建环境。

> **注意**：

> 使用 TinyGo 对版本有限定要求，目前经大规模验证稳定的版本组合是：tinygo 0.29 + go 1.20，可以参考这个官方 [Makefile](https://github.com/alibaba/higress/blob/main/plugins/wasm-go/Makefile)
>
> go 1.24 已经原生支持编译 wasm 文件，相关文档补充中


下面是本地步骤构建 [request-block](https://github.com/alibaba/higress/tree/main/plugins/wasm-go/extensions/request-block) 插件的例子。

### step1. 编译 wasm

```bash
tinygo build -o main.wasm -scheduler=none -target=wasi -gc=custom -tags='custommalloc nottinygc_finalizer' ./main.go
```

详细的编译说明，包括要使用更复杂的 Header 状态管理机制，请参考[ Go 开发插件的最佳实践](https://higress.cn/docs/latest/user/wasm-go/#3-%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90-wasm-%E6%96%87%E4%BB%B6)。

### step2. 构建并推送插件的 docker 镜像

使用这份简单的 Dockerfile

```Dockerfile
FROM scratch
COPY main.wasm plugin.wasm
```

```bash
docker build -t <your_registry_hub>/request-block:1.0.0 -f <your_dockerfile> .
docker push <your_registry_hub>/request-block:1.0.0
```

## 生效 Wasm 插件

### 使用 Higress 控制台

在插件市场中创建点击`创建`按钮，就可以创建自定义插件，在镜像地址一栏中填入上面构建出的 Wasm 镜像地址即可；

创建完成后，点击插件卡片的配置按钮，填入插件的配置（如果有），打开开启开关就生效了。

如果插件逻辑发生了变更，可以构建一个新的镜像，并使用不同的镜像 tag，点插件卡片右上方菜单中的`编辑`按钮，将 Wasm 镜像地址修改为新版本的地址即可。

### 使用 CRD

编写 WasmPlugin 资源如下：

```yaml
apiVersion: extensions.higress.io/v1alpha1
kind: WasmPlugin
metadata:
  name: request-block
  namespace: higress-system
spec:
  defaultConfig:
    block_urls:
    - "swagger.html"
  url: oci://<your_registry_hub>/request-block:1.0.0  # 之前构建和推送的 image 地址
```

使用 `kubectl apply -f <your-wasm-plugin-yaml>` 使资源生效。
资源生效后，如果请求url携带 `swagger.html`, 则这个请求就会被拒绝，例如：

```bash
curl <your_gateway_address>/api/user/swagger.html
```

```text
HTTP/1.1 403 Forbidden
date: Wed, 09 Nov 2022 12:12:32 GMT
server: istio-envoy
content-length: 0
```
