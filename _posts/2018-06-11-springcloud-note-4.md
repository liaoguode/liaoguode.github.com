---
layout: post
title:  "springcloud笔记四（如何使用zuul的filter）"
categories: spring
tag: springcloud
---

使用zuul的filter记录log和处理token
========

记录请求和响应的log
----

1. 记录请求的log

    对于每个请求来说，需要打印请求body中的数据。对于mediatype为application/json的数据，请求中的数据是保存在输入流中，请求中的输入流一旦读取后就被清空了，因此在读取请求后需要再次构建一个带有一样的数据的请求，设置到请求的上下文中。里面比较需要注意的有使用spring的streamutil来读取流，不需要自己写流的读取逻辑，通过改写HttpServletRequestWrapper来将读取出来的流数据再次写入到请求。


2. 打印请求具体处理

```java

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.http.ServletInputStreamWrapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;
import org.springframework.util.StreamUtils;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;

@Component
public class LogRequestFilter extends ZuulFilter {

    private final static Logger log= LoggerFactory.getLogger(LogRequestFilter.class);

    @Override
    public boolean shouldFilter() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request=ctx.getRequest();
        if(request.getRequestURI().equals("/auth/login_validate")){
            return false;
        }
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        try {
            InputStream in = ctx.getRequest().getInputStream();
            HttpServletRequest request=ctx.getRequest();
            String body= StreamUtils.copyToString(in,Charset.forName("UTF-8"));
            log.info(body);
            final byte[] reqBodyBytes=body.getBytes();
            ctx.setRequest(new HttpServletRequestWrapper(request){

                @Override
                public ServletInputStream getInputStream() throws IOException{
                    return new ServletInputStreamWrapper(reqBodyBytes);
                }

                @Override
                public int getContentLength(){
                    return reqBodyBytes.length;
                }

                @Override
                public long getContentLengthLong(){
                    return reqBodyBytes.length;
                }
            });
        } catch (IOException e) {
            log.error("request log print error",e);
        }
        ctx.setSendZuulResponse(true);
        ctx.setResponseStatusCode(200);
        ctx.set("isSuccess", true);
        return null;
    }

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SERVLET_30_WRAPPER_FILTER_ORDER - 2;
    }

}

```

3. 响应处理逻辑

    对于响应来说，和请求碰到的问题是一样的。读取响应的数据后，再次读取将为空。所以对于响应的处理逻辑和请求的处理逻辑是一致的。需要注意的时需要在处理响应的逻辑上加入处理token的逻辑，包括token的失效（toekn黑名单）和token的刷新等。

4. 打印响应实际处理
```java
import com.alibaba.fastjson.JSONObject;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.pccc.aos.gateway.util.JwtUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;
import org.springframework.util.StreamUtils;
import java.io.InputStream;
import java.nio.charset.Charset;

@Component
public class LogResponseFilter extends ZuulFilter {

    private final static Logger log= LoggerFactory.getLogger(LogResponseFilter.class);

    @Override
    public boolean shouldFilter() {
        RequestContext ctx = RequestContext.getCurrentContext();
        int responseStatusCode = ctx.getResponseStatusCode();
        if(responseStatusCode==499){
            return false;
        }
        return true;
    }

    @Override
    public Object run() {
        JwtUtil jwtUtil=new JwtUtil();
        RequestContext ctx = RequestContext.getCurrentContext();
        InputStream out = ctx.getResponseDataStream();
        String outBody=null;
        try {
            outBody = StreamUtils.copyToString(out, Charset.forName("UTF-8"));
            if (outBody != null) {
                log.info(outBody);
                JSONObject json=JSONObject.parseObject(outBody);
                String token = json.getString("token");
                json.put("token", jwtUtil.refreshToken(token));
                String newbody = json.toString();
                ctx.setResponseBody(newbody);
            }
        } catch (Exception e) {
            log.error("response handle error",e);
            ctx.setResponseBody(outBody);
        }
        return null;
    }

    @Override
    public String filterType() {
        return FilterConstants.POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SEND_ERROR_FILTER_ORDER+1;
    }

}

```

5. token处理逻辑
    对于前后端的分离模型来说，后端是完全无状态的，对于用户信息和权限认证来看，一般的处理就是分布式会话或者jwt的token。对比两种实现方案，jwt的token实现更轻量更优美。每个请求都会带有签名的token，网关在处理每个请求时使用对应的处理filter处理token，包括token的解析和认证。

6 token实际处理

```java
import com.alibaba.fastjson.JSONObject;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.http.ServletInputStreamWrapper;
import org.pccc.aos.gateway.util.JwtUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;
import org.springframework.util.StreamUtils;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;

@Component
public class LoginTokenFilter extends ZuulFilter {

    private final static Logger log= LoggerFactory.getLogger(LogRequestFilter.class);

    @Override
    public boolean shouldFilter() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request=ctx.getRequest();
        if(request.getRequestURI().equals("/aos/auth/login_validate")){
            return false;
        }
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        JwtUtil jwtUtil=new JwtUtil();
        try {
            InputStream in = ctx.getRequest().getInputStream();
            String body= StreamUtils.copyToString(in,Charset.forName("UTF-8"));
            JSONObject json=JSONObject.parseObject(body);
            String token=json.getString("token");
            json.put("username",jwtUtil.getUserAccountFromToken(token));
            String newbody=json.toString();
            final byte[] reqBodyBytes=newbody.getBytes();
            ctx.setRequest(new HttpServletRequestWrapper(request){
                @Override
                public ServletInputStream getInputStream() throws IOException{
                    return new ServletInputStreamWrapper(reqBodyBytes);
                }
                @Override
                public int getContentLength(){
                    return reqBodyBytes.length;
                }

                @Override
                public long getContentLengthLong(){
                    return reqBodyBytes.length;
                }
            });
            if (jwtUtil.validateToken(token)) {
                ctx.setSendZuulResponse(true);
                ctx.setResponseStatusCode(200);
                ctx.set("isSuccess", true);
                return null;
            } else {
                ctx.setSendZuulResponse(false);
                ctx.setResponseStatusCode(401);
                ctx.setResponseBody("{\"result\":\"token is not correct!\"}");
                ctx.set("isSuccess", false);
                return null;
            }
        } catch (Exception e) {
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(499);
            ctx.setResponseBody("{\"result\":\"token validate error!\",\"rescode\":\"0\"}");
            ctx.set("isSuccess", false);
            log.error("token validate error",e);
            return null;
        }
    }

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SERVLET_30_WRAPPER_FILTER_ORDER + 2;
    }

}

```