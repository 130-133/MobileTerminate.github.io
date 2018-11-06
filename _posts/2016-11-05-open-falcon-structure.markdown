---
bg: "coding.png"
layout: post
title:  "open-falcon 架构"
crawlertitle: "open-falcon 架构"
summary: "open-falcon 架构"
date:   2018-11-05 10:09:47 +0700
categories: posts
tags: 'open-falcon'
author: leo
---
公司最近要用小米的`open-falcon`做监控，今天对其架构进行了一番了解。
![open_falcon_structure](/assets/images/open_falcon.png)

基本组成模块：
-----------
1. falcon-agent:**数据采集组件**。接收客户端用户数据上报，上报给transfer处理程序。
2. transfer:**数据中转模块**。负责将agent发来的数据转发到judge和graph。
3. judge:**告警判断模块**。根据规则判定收到的数据是否达到报警阈值，若达到则触发报警。
4. alarm:**告警模块**。发出各种渠道的报警消息。
5. graph:**数据存储模块**。上报数据的存储、归档。并提供查询接口
6. query:**查询模块**。给外部提供查询graph中的http接口。
7. dashboard:**前端查询管理页面**，通过query提供的几口查询上报数据。
8. web portal:**配置告警策略模块**。
9. heartbeat server:提供心跳服务及策略缓存。

数据上报及处理流程：
-----------
1. 客户端通过http上报数据给agent
2. agent接收到客户端上报的数据，并通过长连接将数据上报给transfer
3. transfer将收到的数据分别转发给judge集群和graph集群，使用一致性hash做数据分片，一个judge实例或一个graph实例值处理一部分数据
4. 如果judge中判定到了需要告警的事件，则会将该事件推入Redis队列
5. alarm从Redis队列中取告警事件，将告警消息发送给对应渠道
6. graph将收到的数据存储并归档，其提供查询接口给query模块
7. query模块提供外部接口给web前端的dashboard，用于查询graph中存储的上报数据
8. web portal模块用于配置监控策略，并将策略存储到数据库中
9. heartbeat server为心跳服务器，agent每分钟都会发送心跳给heartbeat server，并上报自己的版本，hostname，ip等信息。
10. agent需要获取监控策略中的插件、特殊采集项、IP白名单下发等数据，judge集群在每次进行告警判定的时候也需要从监控策略中拉取判定条件。由于judge实例可能比较多，如果每次都去连接web portal db的话会给db造成比较大的压力。因为agent是通过heartbeat server去web portal db中去获取数据的，因此将heartbeat server作为了web portal db的一层cache，使得judge每次进行告警判定的时候都从heartbeat server获取告警策略，极大的减轻db的压力。

以上就是open-falcon的整体架构啦。
附上[官方视频教程链接][mv-link]

*****

最后，由于官方给出的搭建open-falcon的文档有一些坑，故此给出笔者验证过可用的[open-falcon搭建教程][set-up-open-falcon]

[mv-link]:https://www.jikexueyuan.com/course/1651_1.html?ss=1
[set-up-open-falcon]:https://www.cnblogs.com/straycats/p/7199209.html
