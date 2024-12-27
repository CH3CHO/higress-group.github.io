---
title: "政采云业务网关实践——使用Higress统一替代APISIX/Kong/Istio Ingress"
description: "政采云业务网关实践——使用Higress统一替代APISIX/Kong/Istio Ingress"
date: "2024-12-27"
category: "case"
keywords: ["Higress"]
authors: "CH3CHO"
---

> 作者简介：<font style="color:rgb(0, 0, 0);">政采云基础架构团队技术专家 朱海峰(片风)</font>
>

### 业务网关项目背景
由于一些历史的背景，政采云平台在网关建设上遇到一些问题：

+ 容器网关配置较多，配置方式多样，运维压力较大

配置多是因为容器网关配置分为服务路由、搭建类路由、return/rewrite类路由不同类型的路由。微服务架构使得服务数目多，搭建类平台的技术方案导致子域名非常多，网关的配置复杂度就是MXN(M是服务个数，N是域名的个数)，比如子域名个数约400多，服务个数约500多，整个配置量约20w+；搭建类平台子域名单独定义根路径转发，每个页面的分发路径随意填写，导致网关的配置需要支持到每个搭建页面到路径的映射关系配置；业务的return/rewrite逻辑很多是业务服务迁移过程中产生的，旧的请求一直没有推动下线。一些网关规则配置以服务为中心管理，一些网关规则配置以域名为中心管理，各个服务的路由不唯一，经常会出现配置了网关规则不生效，运维需要排查哪里的配置冲突导致的该问题，并可能出现因一次变更影响其他路由规则的生效问题。

+ 网关技术栈比较多

流量网关使用的是openrestry, 容器网关使用了istio ingressgateway, 开放平台/业务网关使用了spring cloud gateway， 旧的开放平台使用的是kong，搭建网关对应的node express, 跨网网关(apisix)等。虽然不同场景的网关对应不同开发团队，但不同的技术栈选型导致日常维护的工作量增大，同时也给后续的能力建设带来很多的问题。

+ 业务诉求

随着业务发展，业务研发团队对于网关有一些可扩展的能力诉求，比如说认证、安全管控、请求体修改的需求等。



基于上述问题，今年年初政采云有限公司研发中心基础架构组启动了业务网关项目，在项目启动初期，基础架构工程师做了一次需求调研，最终总结出如下诉求：

1）网关配置难题治理。网关配置治理需要解决配置难、配置风险大的问题。

2）网关技术栈尽可能统一。基于降本提效的背景，统一技术栈有利于运维效率的提升；网关技术栈统一就要求网关的能力需要尽可能满足目前大部分业务场景，目前的业务场景有请求转发、请求重写、协议转换、插件扩展(比如之前的业务网关对接了阿里云的人机校验插件)等， 且可持续发展，比如目前能看到的安全管控、流量治理、流量染色等能力。



基础架构团队的工程师经过一个月的需求调研、方案测试，最终给出了解决方案：

+ 网关配置治理
    - 强化规范。基于目前的业务场景，以及后续可能需要支撑的业务场景，优化了域名使用规范，网关规则配置规范，并集成到日常的变更流程，强化规范的落地。
    - 网关规则需要分层配置 。网关配置分层基于不同工程师的负责范畴进行分层，分为域名层和服务层两部分。运维工程师负责域名层的配置，研发工程师负责服务层的配置。
    - 网关规则的管理以应用为中心。应用需要涉及到资源，是有上线、下线等生命周期管理的，把服务层的网关规则跟应用绑定，即在元数据中心里面服务层网关规则就是应用的一个属性。服务层网关规则的生命周期就跟应用是一致的了。
+ 网关技术栈的统一。网关技术栈统一就涉及到网关服务的选型。

### 网关服务的选型
技术栈统一是逐步达成的目标，不过从目前需要解决的问题紧急度上看，容器网关、业务网关需要合并，以一个技术栈解决网关配置治理、扩展能力的问题。经过社区活跃度、能力对比等多个维度的比较，最终筛选出apisix、higress、istio(ingressgateway)这三个网关服务。

#### higress vs istio(ingressgateway)
higress基于istio的能力做了更上层的封装，以提高易用性，以及满足更多业务场景的需求。基于Higress, 我们可以获取如下收益:

+ 插件能力加强。higress在开源的wasm client上做了封装，更方便开发者做插件的开发，比如做了插件生效的范围控制，并且社区已有很多插件能力可以直接使用。
+ 扩展功能。higress在多数据源这块集成了多种主流注册中心，这些能力可以直接复用，不用重复建设；基于envoyfilter提供了http2rpc的能力，并封装了单独的CRD, 方便http2dubbo的配置。

#### apisix vs higress
apisix是一个比较优秀的云原生网关解决方案，底层基于openresty, 且在公司高速公路项目上也使用了apisix做跨网络域的请求转发。不过考虑到目前网关重度使用场景是容器网关，该网关规则对应的是virtualservice资源，我们也有内部系统围绕virtualservice做了相关解决方案，原生支持virtualservice会简化迁移工作量, 我们选择了higress。

#### 为什么使用Istio CRD
在技术选型的时候，我们不断的思考一个问题，为什么继续使用Istio CRD, 而不是使用kubernetes Ingress或者gateway API做业务网关的配置sheme， 选择了标准的规范会不会简化后续的网关能力建设?

kubernetes Ingress提供了基础的路由能力，不过与我们的使用场景相比还是简单了些，如果把我们网关的配置复制到kubernetes Ingress的描述方式，会存在大量的注解信息配置，以及整体配置会膨胀很多倍。因为Ingress定义每个ingress资源只能配置一个Host, 如果要多个host就需要对应多个Ingress资源，而istio CRD里面就一个virtualservice资源，配置简洁，可读性较强。

随着业务的发展，网关侧的能力要求越来越高，比如流量治理、插件扩展能力、mesh能力等, ingress满足不了我们对网关配置的要求。

kubernetes社区定义了gateway API, 以解决Ingress目前存在的问题。不过目前gateway API发布了1.1版本，能力的丰富度上还不是不如istio CRD, 并且gateway API在低版本的kubernetes上存在兼容性问题。

最终我们继续选择istio CRD做网关配置的资源。

### 社区的支持
![](https://cdn.nlark.com/yuque/0/2024/png/1332437/1722096034627-befaff47-bb6c-48f6-a4aa-d79ba88e0bb6.png)

Higress架构图



从Higress架构图看，higress数据面还是使用envoy做请求转发，使用istio pilot组件做配置发现，istioCRD是istio默认支持的资源，使用higress+istioCRD理论上是很顺利的。

迁移过程遇到了一些问题，分享出来供参考:

1. <font style="color:rgb(0, 0, 0);">基于域名聚合功能不生效。</font>为了优化变更速度，higress默认开启了<font style="color:rgb(0, 0, 0);">enableSRDS，该特性会按照域名隔离生效到envoy里面的配置，因为我们业务场景下一个路由对应的子域名特别多，之前是开启了基于域名合并网关配置，在使用higress会导致配置量大增, 关闭了enableSRDS该问题就解决了。</font>
2. <font style="color:rgb(0, 0, 0);">配置生效特别慢。在大量网关配置生效到Higress的时候会达分钟级别。经过higress社区同学排查优化了这块逻辑，最终使用higress配置的生效时间也是秒级的。</font>
3. <font style="color:rgb(0, 0, 0);">在有冲突网关规则下发生效的时候istio出现生效混乱。istio 1.22.0以下的版本都存在一个问题，在有冲突网关规则下发的时候会在特定条件下出现生效混乱的问题, 刚好我这次做网关规则迁移过程中发现了该问题，在我给istio社区提bug的时候，他们刚在1.22.0版本修复该问题。higress社区同学很快的帮忙merge该修复的commit, 并发布在higress1.4.1版本中。</font>
4. <font style="color:rgb(0, 0, 0);">在同样配置下，higress使用内存较高。从监控可以看到，在相同配置下，higress gateway的内存使用明显比istio-ingressgateway要高。经过higress社区同学精简higress gateway的监控指标，两者也达到基本一致的资源使用。</font>

![](https://cdn.nlark.com/yuque/0/2024/png/1332437/1722093797486-77785993-8044-4c14-a9fc-8d8a34781e93.png)

higress gateway 与istio ingressgateway内存占用对比

 问题处理中，higress社区积极响应，基本上都是在问题稳定复现的当天就给出解决方案，配置生效慢问题只是我们场景下遇到的问题，他们本地复现比较困难的情况下，依然坚持在当天解决该问题。在higress社区的大力支持下，问题都迎刃而解，事实证明，higress+istioCRD替换istio整体进度可控。

higress社区同学对于我们提的问题回复效率极高，我们在看higress代码，学习插件开发的时候，经常会有些问题提到higress社区钉钉群中，社区同学看到就会积极回复。

业务网关项目除了引入higress取代ingressgateway作为业务网关外，还重点做了网关规则治理。下面我介绍下网关分层的解决思路，与业务耦合比较紧密，供参考。

### 网关分层的解决思路
在网关规则治理上，结合目前网关规则的分层配置要求，我们定义了应用路由(appvirtualservice, 简称avs)的CRD资源，把路由配置分成两层，域名层和服务层。域名层对应了原来的virtualservice资源，服务层对应了avs资源，avs是不带域名信息的，跟域名是分开的，研发人员操作的网关规则都是对应的avs资源，avs资源最终会生效到virtualservice资源。

![](https://cdn.nlark.com/yuque/0/2024/png/1332437/1722097701381-16f30dac-cc55-4049-9fa2-4c93c9dfce14.png)

人员与资源操作关系

avs资源举例：

```plain
apiVersion: arch.cai-inc.com/v1
kind: AppVirtualService
metadata:
  labels:
    app: web-a-front
    vsLayer: app
  name: web-a-front
  namespace: test
spec:
  http:
  - match:
    - uri:
        prefix: /a-index
    route:
    - destination:
        host: web-a-front
        port:
          number: 80
```

基于实际的使用场景，在avs资源定义中，我们还定义了PriorityClass(支持高，中，低三个等级，默认是中)字段，通过该字段决定该规则生效的优先级，部分缓解原来依赖创建时间决定规则生效优先级的不可控问题。

网关分层需要对原来人工维护的数量接近2000的virtualservice配置进行规范性整理，对整体逻辑影响比较大，为了保证整个过程可重复执行，我们开发了工具，把原来的virtualservice逻辑拆分成virtualservice+avs的两层配置结构。

基于原来的路由规则，我们把路由分为域名默认路由、服务路由(冲突服务路由、不冲突的服务路由)、rr路由(return/rewrite路由)。域名默认路由、rr路由、冲突的服务路由跟域名是有关系的，需要网关运维人员维护处理；不冲突的服务路由(比例占比90%)转换为avs资源，通过页面功能开放给研发人员处理。以此测算，分层前后网关规则的维护成本(问题排查以及出错概率)降低90%。

分层解决了，我们面临一个问题，如何校验变更前后的数据一致性，以及如何做变更？

#### 数据一致性校验
路由变更的数据一致性校验的方式是采集原来流量转发的日志，回放流量到网关分层后的测试环境中，再采集该测试环境的网关转发日志，两者做对比，输出差异报告，如果同一个请求(域名+url相同)，转发到upstream cluster不同，则说明网关分层后修改了原来的转发逻辑，需要优化分层逻辑或者人工处理。

![](https://cdn.nlark.com/yuque/0/2024/png/1332437/1722097054738-461276cf-faeb-436b-a5b5-98af32198212.png)

流量校验逻辑图示

开发工具的时候两个点需要注意：

1）流量唯一性采集 流量采集使用域名+url(去除参数)的md5做唯一性key，对于有请求url路径上带了变化id的，比如/item/123445, 通过正则表达式过滤成一条唯一的请求,  流量明细会记录到文件中，包含流量的域名, url, 后端服务, requestId等字段，用于流量回放以及差异性验证。

2）回放流量使用HEAD请求。回放流量只需要验证转发逻辑的正确，为了避免影响到生产环境的流量，原来的请求method都会设置HEAD, 去掉参数和请求体内容进行回放。

#### 变更方案
 创建新的网关实例，分层之后的网关规则生效到新的网关实例，在负载均衡侧基于流量比例1%->5%->10%->20%->50%->100%的比例逐渐放大新网关实例承载的流量比例。

### 初步建设成果
业务网关项目初步建设的成果还是较显著的：

+ 无感。整个业务网关的变更对业务研发侧基本是无感的，他们只是感知到有这种变更事件。
+ 网关规则治理目标达成。网关规则按照域名层和服务层进行分层，域名层属于基础配置，服务层以应用为中心进行管理，整体网关规则的治理目标基本达成。
+ 技术栈统一。业务网关(higress)取代了容器网关(istio ingressgateway), 旧的开放平台(kong), 跨网网关(apisix), 旧的业务网关(spring cloud gateway)，节省了相关服务所占的资源，同时节省了后续的维护成本。

引入higress目前仅仅是开始，后续会基于higress建设开放平台(认证鉴权、http转dubbo等)、接入统一认证能力等，最终达成统一网关技术栈的目标。

