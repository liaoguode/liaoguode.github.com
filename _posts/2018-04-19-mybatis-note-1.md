---
layout: post
title:  "mybatis笔记一"
date:  2018-04-19 20:00:00
categories: javaee
auth: peterliao
tag: 
    - mybatis
---

Mybatis注解使用-条件sql
========

* 对于使用mybatis注解的@select等写sql时，注意一点就是当使用动态sql脚本时，一定要在前后加上script标签,并且collection后面可以跟上任意集合名，为@param映射过来的变量名称。如下所示

```java
    @Select("<script>"+
            "select distinct ra.auth_auth_id from aos.role_auth as ra where ra.role_role_id in "+
            "<foreach collection=\"roleIds\" item=\"item\" index=\"index\" open=\"(\" close=\")\" separator=\",\">" +
            "#{item}" +
            "</foreach>"+
            "</script>")
    List<String> findAuthByRoles(@Param(value = "roleIds")int[] roleIds);
```