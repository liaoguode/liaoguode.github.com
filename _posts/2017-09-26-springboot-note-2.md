---
layout: post
title:  "springboot学习笔记二"
categories: spring
tag: springboot
---
spring boot 学习笔记二(数据库操作)
=======

spring boot整合druid
-------

1.druid是什么

* druid是alibaba开源的一个数据库连接池，通过网上的比较发现，性能不错，扩展性强，具有SQL解析和页面监控功能，功能强大。

2.如何整合spring boot 和druid

* 引入maven的[jar包](http://www.mvnrepository.com/artifact/com.alibaba/druid)

``` xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>${druid-version}</version>
</dependency>
```

3.配置druid

* 配置数据源

```properties
#数据库设置
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/ansibleui?useUnicode=true&characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=liao5414
#--------------------------
# 下面为连接池的补充设置，应用到上面所有数据源中
# 初始化大小，最小，最大
spring.datasource.initialSize=5
spring.datasource.minIdle=5
spring.datasource.maxActive=20
# 配置获取连接等待超时的时间
spring.datasource.maxWait=60000
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.timeBetweenEvictionRunsMillis=60000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.minEvictableIdleTimeMillis=300000
spring.datasource.validationQuery=SELECT 1 FROM DUAL
spring.datasource.testWhileIdle=true
spring.datasource.testOnBorrow=false
spring.datasource.testOnReturn=false
# 打开PSCache，并且指定每个连接上PSCache的大小
spring.datasource.poolPreparedStatements=true
spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
spring.datasource.filters=stat,wall,log4j
# 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
```

* 注入数据源配置

```java
package org.ansible.ui.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration  //设置为配置的bean

public class DruidDataSourceConfiguration {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource") //注入配置到bean类,需要搭配@Bean
    public DataSource druidDataSource(){
        DruidDataSource druidDataSource=new DruidDataSource();
        return druidDataSource;
    }
}
```

* 设置druid的filter

```java
package org.ansible.ui;

import com.alibaba.druid.support.http.WebStatFilter;

import javax.servlet.annotation.WebFilter;
import javax.servlet.annotation.WebInitParam;

@WebFilter(filterName = "druidWebStatFilter", urlPatterns = "/*",
        initParams = {
                @WebInitParam(name = "exclusions", value = "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")
        }
)
public class DruidStatFilter extends WebStatFilter {
}
```

* 配置druid监控界面

```java
package org.ansible.ui;

import com.alibaba.druid.support.http.StatViewServlet;

import javax.servlet.annotation.WebInitParam;
import javax.servlet.annotation.WebServlet;

@WebServlet(
        urlPatterns = "/druid/*",
        initParams = {
                @WebInitParam(name="allow",value="127.0.0.1"),
                @WebInitParam(name="delay",value="182.180.0.1"),
                @WebInitParam(name="loginUsername",value="peterliao"),
                @WebInitParam(name="loginPassword",value = "liao5414"),
                @WebInitParam(name="resetEnable",value = "false")
        }
)
public class DruidStatViewServlet extends StatViewServlet {
}
```

4.监控地址
