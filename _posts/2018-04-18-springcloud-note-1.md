---
layout: post
title:  "springcloud学习笔记一"
categories: spring
tag: springcloud
---

Spring Cloud Feign
========

使用说明
-------
    
在spring cloud启动类上加上注解@EnableFeignClients启用Feign的使用，然后还需要加上@EnableDiscoveryClient启用eureka的使用，Feign是整合了ribbon和hystrix之后的组件，具有服务消费和断路器的功能。在使用了Feign后将大大简化服务调用的步骤。

在使用注解启用feign的功能后，启用一个service接口，使用@Feign的注解标注该接口，注解里的值为service的名称，为eureka中注册的服务名，然后在接口的方法里使用@RequestMapping来指定调用服务的那个rest接口。至于服务rest接口的参数注入，和controller请求的参数注入一样，使用@RequestParam,@RequestBody和@RequestHeader来处理,@RequestParam和@RequestHeader的value值不能被省略，对应的value值指定了远程接口的参数名。


使用demo
--------


