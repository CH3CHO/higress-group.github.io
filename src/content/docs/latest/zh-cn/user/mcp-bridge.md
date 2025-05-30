---
title: Mcp Bridge 配置说明
keywords: [discovery]
description: Higress Mcp Bridge 配置说明。
custom_edit_url: https://github.com/higress-group/higress-group.github.io/blob/main/src/content/docs/latest/zh-cn/user/mcp-bridge.md
---

# Mcp Bridge 配置说明

> **注意**

> 当前仅会监听在 higress 安装命名空间下命名为 `default` 的 mcpbridge 资源
>
> 当使用 higress UI 控制台时，此配置的对应入口位于菜单项中的**服务来源**


## McpBridge 字段说明
| 字段         | 类型 | 说明                | 示例值     | 是否必填              |
|------------| --- |-------------------|---------|-------------------|
| registries | RegistryConfig 数组 | 支持配置多个不同注册中心的服务来源 | []      | 否                 |
| name       | 字符串 | McpBridge 名称      | default | 是，当前该值只能是 default |

## RegistryConfig 字段说明
| 字段                    | 类型 | 说明                                           | 示例值                                  | 是否必填 |
|-----------------------| --- |----------------------------------------------|--------------------------------------| --- |
| type                  | 字符串 | 服务发现类型，可选项：nacos,nacos2,nacos3,zookeeper,consul,eureka,static,dns | nacos2                               | 是 |
| name                  | 字符串 | 自定义的服务来源名称                                   | my-nacos                             | 是 |
| domain                | 字符串 | 服务发现地址，一般是注册中心地址; 当类型是static或dns时，这个字段用于直接配置后端地址                             | 192.168.1.2                          | 是 |
| port                  | 整数 | 注册中心访问端口                                     | 8848                                 | 是 |
| protocol                  | 字符串 | 服务的协议，仅对 static/dns 类型生效，支持配置 HTTP/HTTPS/GRPC/GRPCS，默认值是 HTTP                                     | HTTPS                                 | 否 |
| sni                  | 字符串 | 当 protocol 是 HTTPS 或 GRPCS 时，用于设置 TLS 握手时使用的 SNI                                     | www.example.com                                 | 否 |
| zkServicesPath        | 字符串数组 | 使用zk时,填写服务注册的根路径,默认监听 /dubbo 和 /services，前者为dubbo 服务默认根路径，后者为SpringCloud服务默认根路径 | ["/service-provider"]                | 否 |
| nacosNamespaceId      | 字符串 | nacos命名空间id                                  | d8ac64f3-xxxx-xxxx-xxxx-47a814ecf358 | 否 |
| nacosGroups           | 字符串数组 | nacos服务分组列表                                  | ["DEFAULT_GROUP"]                    | 否 |
| nacosAccessKey        | 字符串 | nacos需要认证时填写的ak信息                            | xxxx                                 | 否 |
| nacosSecretKey        | 字符串 | nacos需要认证时填写的sk信息                            | xxxx | 否 |
| nacosRefreshInterval  | 整数 | nacos需要认证时填写的服务刷新间隔纳秒数                       | 30000000000（30秒）                     | 否 |
| mcpServerExportDomains| 字符串数组 | 开启mcp server时要暴露的域名，不填时在所有域名暴露    | ["example.com","a.test.com"]                     | 否 |
| mcpServerBaseUrl      | 字符串 | 开启mcp server时要暴露的url，或者url前缀（例如使用nacos聚合多个mcp server时）  | /mcp-server                     | 否 |
| enableMCPServer       | 布尔值 | 是否开启MCPServer（目前仅支持nacos3类型的服务来源）  | true                     | 否 |
| consulDatacenter      | 字符串 | consul需要认证时填写的数据中心名称                         | dc1                                  | 否 |
| consulServiceTag      | 字符串 | consul需要认证时填写的服务标签过滤                         | higress                              | 否 |
| consulRefreshInterval | 整数 | consul需要认证时填写的服务刷新间隔纳秒数                      | 30000000000（30秒）                     | 否 |
| authSecretName        | 字符串 | 服务发现认证信息 Secret 名称, 具体看下面说明                  | higress-nacos-auth                      | 否 |

## authSecretName 字段说明

服务发现来源认证信息保存在 higress-system 命名空间下对应 Secret 下 ，现在支持 nacos 和 consul 服务发现。

nacos 认证信息包含用户名和密码， consul 认证信息包含认证 token， 具体 Secret 配置样例如下：

### nacos 认证

```yaml
apiVersion: v1
data:
  nacosPassword: aGlncmVzcw==
  nacosUsername: aGlncmVzcw==
kind: Secret
metadata:
  name: higress-nacos-auth
  namespace: higress-system
type: Opaque
```

### consul 认证 

```yaml
apiVersion: v1
data:
  consulToken: NGFkZDE2NjUtNTM5NC03YzU1LTIzYTUtNzA4MDZkNTFiYzQy
kind: Secret
metadata:
  name: higress-consul-auth
  namespace: higress-system
type: Opaque
```


## 支持配置静态的服务发现方式

### 通过固定IP发现服务

```yaml
apiVersion: networking.higress.io/v1
kind: McpBridge
metadata:
  name: default
  namespace: higress-system
spec:
  registries:
  - domain: "1.1.1.1:80,2.2.2.2:80"
    name: test
    port: 80
    type: static
```

### 通过DNS域名发现服务

```yaml
apiVersion: networking.higress.io/v1
kind: McpBridge
metadata:
  name: default
  namespace: higress-system
spec:
  registries:
  - domain: www.alibaba.com
    name: alibaba
    port: 443
    protocol: HTTPS
    type: dns
```
