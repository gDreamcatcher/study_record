# Knative

## 概念
Knative是一款基于Kubernetes的Serverless框架。其目标是制定云原生、跨平台的Serverless编排标准。Knative通过整合容器构建（或者函数）、工作负载管理（动态扩缩）以及事件模型这三者来实现的这一Serverless标准。
Knative API建立在现有Kubernetes API的基础上，因此Knative资源与其他Kubernetes资源兼容，并且可以由集群管理员使用现有Kubernetes工具进行管理。


## Knative Server
Knative server用于帮助开发者轻松管理Kubernetes上的无状态服务，使开发者不用考虑服务的自动扩缩容、联网和部署问题。

## Knative Event
通过将路由规则放到配置文件中而不是嵌入到代码里，从而将事件在线上集群和离线集群之间路由变得更简单。

