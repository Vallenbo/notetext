# OpenTelemetry链路追踪

[OpenTelemetry使用文档](https://opentelemetry.io/docs/)

OTel解决分布式链路追踪标准不统一的问题

1. OpenTelemetry是什么
2. openTelemetry能做什么
3 、基于OTel标准的Jaeger与zipkin接入
4 、基于OTel标准接入prometheus
5. baggage统一数据存储与传播

微服务:
分布式链路追踪技术的标准，数据格式，系统指标
zipkin（java）, jaeger（golang）分布式链路追踪

jaeger ,zipkin
server
agent
dbStor
go-client

jaeger , zipkin opentracing 标准链路追踪指标
prometheus其他监控metrics，OpenCensus 标准监控指标
opentracing + OpenCensus = OpenTelemetry链路追踪和指标

# opentelemetry能做什么

1 、每种语言都提供了响应的lib库
2 、提供了中立数据采集器，并支持多种方式部署
3 、生成、发送、收集、处理和导出遥测数据
4 、可以通过配置将数据并行发送到多个目的地
5 、提供了Opentracing和OpenCensus垫片



traces链路，请求的调用链，同- -链路的所有
span具有相同的trace_id
span跨度，一个完整逻辑，是一个span。一个方法是一个span



