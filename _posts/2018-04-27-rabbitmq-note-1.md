---
layout: post
title:  "rabbitmq笔记一"
date:  2018-04-19 20:00:00
categories: mq
auth: peterliao
tag: 
    - rabbitmq
---

command line tool
========

rabbitmq命令工具有三个，分别为rabbirmqctl、rabbitmq-plugins、rabbitmqadmin。不同的工具有不同的使用场景，rabbitmqctl只能被rabbitmq节点的管理员使用，包括virtual host,user，permission和节点数据的管理。rabbitmq-plugins用于管理rabbitmq的插件。rabbitmqadmin构建在http api上，需要开通http api端口，并且依赖python。

rabbitmq依赖erlang，rabbitmq节点间的通信验证机制使用erlang的cookie。需要系统使用编码为utf-8。

