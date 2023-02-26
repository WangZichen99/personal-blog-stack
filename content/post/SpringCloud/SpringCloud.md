---
title: SpringCloud 
slug: SpringCloud 
author: wzc 
date: 2022-11-26T11:52:46+08:00 
categories:
  - 学习笔记 
tags:
  - Spring 
  - Spring Cloud 
image: img/year/month/pic_name.jpg 
draft: true

---

# Spring Cloud

## 微服务概述

### 什么是微服务

**微服务**是一个解决方案是一个服务应用，也就是工程中一个一个的模块。

**微服务架构**是一种架构模式或风格，它是将单一的应用程序划分为多个小的服务， 服务之间采用轻量级的通信协议（HTTP）进行通信，并且服务之间能够独立的部署到生产环境。
对于每个服务应该根据业务需求选择合适的语言和工具进行构建，最后使用轻量级的集中式管理协调这些服务。

### 微服务的优缺点

#### 优点

1. 每个服务足够小，高内聚
2. 开发简单，一个服务对应一个职责
3. 低耦合，每个服务能够使用不同的语言进行开发
4. 服务的开发和部署都是互相独立的
5. 容易集成第三方工具
6. 微服务更适合小型团队开发和维护

#### 缺点

1. 分布式系统复杂性
2. 运维难度随着服务数量而增加
3. ...

## SpringCloud入门概述

### SpringCloud是什么

SpringCloud基于SpringBoot提供了一套微服务解决方案，包括服务注册中心、配置中心、全链路监控、服务网关、负载均衡、熔断器等等。
SpringBoot没有重复造轮子，而是将各种成熟的服务框架重新封装起来供开发者调用。SpringCloud是分布式微服务架构的一站式解决方案。

### SpringCloud和SpringBoot的关系

- SpringBoot专注于开发单个微服务，而SpringCloud关注的是多个微服务之间的通信、协调、注册等任务，是微服务协调治理的框架
- SpringBoot可以独立使用，但SpringCloud必须依托于SpringBoot

### Dubbo和SpringCloud技术选型

|        |     Dubbo     |            Spring            |
|:------:|:-------------:|:----------------------------:|
| 服务注册中心 |   Zookeeper   | Spring Cloud Netfilx Eureka  |
| 服务调用方式 |      RPC      |           REST API           |
|  服务监控  | Dubbo-monitor |      Spring Boot Admin       |
|  断路器   |      不完善      | Spring Cloud Netflix Hystrix |
|  服务网关  |       无       |  Spring Cloud Netflix Zuul   |
| 分布式配置  |       无       |     Spring Cloud Config      |
|  服务跟踪  |       无       |     Spring Cloud Sleuth      |
|  消息总线  |       无       |       Spring Cloud Bus       |
|  数据流   |       无       |     Spring Cloud Stream      |
|  批量任务  |       无       |      Spring Cloud Task       |

最大的区别：Spring Cloud抛弃了RPC通信，采用的是基于HTTP的REST方式

SpringCloud在配置、封装、上手难易以及社区环境都要比Dubbo更加容易，SpringCloud就像一台品牌机，Dubbo更像是一台组装机

Dubbo的定位是一款RPC框架，Spring Cloud的目标才是微服务架构























