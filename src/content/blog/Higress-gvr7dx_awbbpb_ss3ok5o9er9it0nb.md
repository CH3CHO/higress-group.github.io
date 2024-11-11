---
title: "应用网关的演进历程和分类"
description: "应用网关的演进历程和分类"
date: "2024-11-06"
category: "article"
keywords: ["Higress"]
authors: "CH3CHO"
---

<font style="color:rgb(0, 0, 0);">唯一不变的是变化，在现代复杂的商业环境中，企业的业务形态与规模往往处于不断变化和扩大之中。这种动态发展对企业的信息系统提出了更高的要求，特别是在软件架构方面。为了应对不断变化的市场需求和业务扩展，软件架构必须进行相应的演进和优化。网关作为互联网流量的入口，其形态也在跟随软件架构持续演进迭代中。我们下面就聊一聊网关的演进历程以及在时下火热的AI浪潮下，网关又会迸发怎样新的形态。</font>

# <font style="color:rgb(0, 0, 0);">网关演进形态概览</font>
软件架构的演进是一个不断适应技术发展和业务需求变化的过程，伴随着软件架构的演进网关的形态也在随之持续迭代，在不同软件架构阶段中网关也呈现其不同的形态。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/34709/1729233474984-5ed608d9-0bc8-4f63-96a2-6b9221018185.png)

软件架构的演进是技术发展和业务需求不断推动的结果，从早期的简单设计到如今复杂的多层次架构，体现了软件系统在可扩展性、维护性和性能方面的不断追求。

1. 单体架构：在软件工程的初期，单体架构是最常见的形式。所有的功能模块都集成在一个整体应用中。这种架构的优点是初期开发和部署比较简单，但随着系统功能的扩展和复杂性增加，维护和升级变得越来越困难，单点故障的风险也越来越高。
2. 垂直架构：为了解决单体架构带来的维护问题，逐渐发展出垂直架构。不同的业务功能被拆分成不同的模块，每个模块独立开发和部署。虽然这种架构在一定程度上提升了效率，但模块之间的依赖关系依然紧密，扩展和协作问题仍然存在。
3. SOA架构（面向服务的架构）：SOA 架构通过服务松耦合来解决垂直架构中的依赖问题。系统被设计成一组相互协作的微服务，每个服务通过定义明确的接口进行通信。这种方式提高了系统的灵活性和可扩展性，但管理和运维复杂度增加，需要采用成熟的服务治理框架。
4. 微服务架构：微服务架构是 SOA 架构的发展和细化，它将应用进一步拆分成更小的服务单元，每个单元可以独立开发、部署和扩展。微服务架构强调容错和自动化运维，适合大规模、高复杂度的系统。但其实施也需要更复杂的测试、监控和管理工具支持。
5. 云原生架构：云原生架构进一步解放了开发者，允许他们专注于代码逻辑，而不需要关心底层的基础设施。通过统一的 K8s 运维底座，系统的扩展和缩减可以更加灵活地自动进行。云原生架构带来了更高的弹性和资源利用率，同时也要求开发者适应新的开发和运维模式。
6. AI原生架构：随着人工智能和机器学习技术的发展，AI 原生架构应运而生。这种架构将大规模语言模型集成到应用系统中，提供自然语言处理、智能推荐等功能。AI 原生架构不仅改变了传统应用的交互方式，还带来了更智能的业务决策支持。但这种架构也对计算资源和数据隐私提出了更高的要求。

总体来看，软件架构演进的每一步都在寻求解决当前架构的瓶颈和不足，通过技术创新和最佳实践，使得软件系统能够更好地应对不断变化的业务需求和技术环境。

# 网关演进之流量网关
流量网关作为网络架构中的关键组件，主要负责管理和优化数据流量，以提升业务的可伸缩性和高可用性。 Nginx 作为流量网关的代表性软件，以其高效的性能和灵活的配置广受欢迎。流量网关的核心目的是解决多业务节点的流量负载均衡问题，通过智能调度将客户请求分配到不同的服务器上，从而均匀分摊负载，避免单点故障，确保服务的稳定性和连续性。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/34709/1729233508468-5926f272-d9d8-49e4-8432-455766c33833.png)

# 网关演进之ESB
企业服务总线（ESB）网关是一个专为企业设计的关键集成解决方案，旨在标准化和简化不同系统和服务之间的通信与消息传送。作为核心通信基础设施，ESB网关可以减少系统间的耦合性，提高互操作性和灵活性，确保数据和服务的无缝整合。遵循服务导向型架构（SOA）原则，ESB 通过集中管理消息路由、转换和安全，实现服务的快速部署和高效运作。它支持不同协议和数据格式，提升了系统的扩展性和可维护性，帮助企业在不断变化的业务环境中保持竞争力与创新。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/34709/1729233537869-cdac8d19-e4ad-491f-9b95-13e68865d871.png)

# 网关演进之微服务网关
微服务网关是微服务架构中的关键组件，负责集中管理微服务的路由规则，增强系统安全性，提供性能监控，并简化访问流程，从而提高整个系统的可靠性。微服务网关可以实现负载均衡、限流、熔断、身份验证等功能，通过统一入口管理和优化各微服务间的交互。此举不仅简化了客户端与微服务的通信复杂性，还为系统安全提供了额外的保护，Spring Cloud Gateway 是一个广泛应用的微服务网关，它基于 Spring 生态系统，易于与 Spring Boot 项目集成，因其灵活、高效和可扩展性受到了开发者的青睐。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/34709/1729233574600-d293562e-721a-4560-bc3e-c4b63bb1b74b.png)

# 网关演进之云原生网关
云原生网关是伴随 K8s 的广泛应用而诞生的一种创新网关，K8s 集群内外网络天然隔离的特性要求通过网关来将外部请求转发给集群内部服务，K8s 采用 Ingress/Gateway API 来统一网关的配置方式，同时 K8s 提供了弹性扩缩容来帮助用户解决应用容量调度问题，基于此用户对网关产生了新的诉求：期望网关既能有流量网关的特性来处理海量请求，又具备微服务网关的特性来做服务发现与服务治理，同时要求网关也具备弹性扩缩容能力解决容量调度问题，能够让开发者能够专注于业务逻辑的实现，而无需担心底层架构的容量、维护和管理。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/34709/1729233603767-cf2f079f-6286-4e26-a6d8-cb6b0b748588.png)

# 网关演进的下一站：AI网关
AI 网关是专为处理 AI 流量设计的关键组件，具备长连接、大带宽、高延时的特性，能够高效管理和优化数据传输。在 AI 应用、AI 平台和大型语言模型（LLM）中，不同的场景对 AI 网关的性能和功能有着多样化的需求。为更好地满足这些需求，AI 网关提供了丰富的AI插件集，开发者可以通过低代码方式轻松集成和构建复杂的 AI 应用。这不仅大幅降低了开发难度，还显著提高了效率，使得 AI 技术在各类应用和平台中更加易用和普及。AI 网关因此成为推动AI创新和应用落地的核心推动力。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/34709/1729233632979-97c6c19e-c767-48cb-b692-0b0b5a7f8b4c.png)

# API网关去哪了？
读到这里的同学可能会有这么一个问题：API 网关去哪了？我们耳熟能详的API网关为什么没有提到呢？回答这个问题之前我们可以先问自己一个问题：API 是什么？API 包含什么？

在流量网关中我们配置的路由本身也是一种API，只是没有定义规范的请求/响应标准而已，通常我们称为 HTTP API，在 AWS API Gateway 中的 HTTP API 指的就是这个，目前 HTTP API 使用最简单、应用最广；REST API 相信大家基本都或多或少听说过，它采用 JSON 格式来规范请求/响应，在微服务网关中应用较广；Websocket/gRPC/Dubbo API这 些相信大家同样不会陌生，因此可以说支持 API 访问的都是 API 网关，API 网关是贯穿软件架构演进的各个阶段。**时间来到了当下，在 AI 浪潮下 API 网关又会迸发出怎样新的特性呢？**

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/34709/1729233665717-d44614d3-4787-42ee-98b4-920e9f5556e7.png)

# Higress：AI 原生的 API 网关
Higress 源自阿里巴巴内部电商、交易等核心生产场景的实践沉淀，遵循 Ingress/Gateway API 标准，将流量网关、微服务网关、安全网关三合一，并在此基础上扩展了服务管理插件、安全类插件和自定义插件，高度集成 K8s 和微服务生态，包括 Nacos 注册和配置、Sentinel 限流降级等能力，并支持规则变更毫秒级生效等热更新能力。

在 AI 浪潮下，Higress 面向 AI Native 场景提供原生扩展能力以满足复杂的 AI 应用需求。它不仅支持常见的API管理、流量控制和安全策略，还具备与 AI 模型无缝集成的特性。Higress通过灵活的插件机制，允许开发者轻松添加自定义功能，提升扩展性和适应不同应用场景的能力。借助高性能的处理能力和智能流量调度，Higress 网关可以显著减少延迟，提升 AI 服务的响应速度。此外，Higress 强大的监控和日志功能，帮助开发者快速定位和解决问题，使 AI 应用的开发和运维变得更加高效和可靠。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/34709/1729233698411-8298d0cb-32c8-4a36-b0e4-3c9ab0682e7b.png)

<font style="color:rgb(0, 0, 0);"></font>

<font style="color:rgb(0, 0, 0);">本文整理自《统一多层网关架构视频课程》的第一期视频，下载课程PPT和查看回放请点击</font>[下载课程 PPT ](https://developer.aliyun.com/ebook/8395)<font style="color:rgb(0, 0, 0);">或</font>[查看回放](https://developer.aliyun.com/live/254581)<font style="color:rgb(0, 0, 0);">。</font>

# 更多课程
![](https://intranetproxy.alipay.com/skylark/lark/0/2024/jpeg/299576/1730874857377-bda06858-48cd-49e1-8a77-d9865f490f87.jpeg)

