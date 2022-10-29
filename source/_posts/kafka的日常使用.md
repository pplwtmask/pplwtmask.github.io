---
title: kafka的日常使用
date: 2022-10-29 15:38:06
---



## kafka集群

![](/img/kafka.png)


* 每个broker就是一个kafka-server，每个broker可以分布在不同的机器上
* Partition 分布式存在多个broker上，每个partition可以配置多个replica，一主（leader）多从（followers）


kafka实际使用中需要注意的几点
>1. Kafka只保证在一个Partition内部，消息是有序的，但是，存在多个Partition的情况下，Producer发送的3个消息会依次发送到Partition-1、Partition-2和Partition-3，Consumer从3个Partition接收的消息并不一定是Producer发送的顺序，因此，多个Partition只能保证接收消息大概率按发送时间有序，并不能保证完全按Producer发送的顺序
2. Group ID相同的多个Consumer实际上被视作一个Consumer，即如果有两个Group ID相同的Consumer，那么它们各自收到的很可能是A、C和B、D（假设Producer发送了A、B、C、D四条消息）
