---
layout: post
title:  "python笔记一（如何离线部署一个flask工程）"
date:  2018-06-11 20:00:00
categories: python笔记
auth: peterliao
tag: 
    - python笔记
---

如何离线部署一个flask工程
====

离线安装pip
----


1. 在官网（https://pypi.python.org/pypi/setuptools#code-of-conduct）下载对应的setuptools的包

2. 解压下载的包，执行python setup.py install安装setuptools

3. 在官网(https://pypi.python.org/pypi/pip)下载对应的pip的包

4. 解压下载的包，执行python setup.py install安装pip

下载依赖
----

1. 确认依赖包名称

2. 通过在生产环境同样的操作系统版本下使用pip下载相应的依赖包，执行pip install -d <dir> packagename1 packagename2 ........下载到对应的目录下

3. 将包上传到离线的服务器上，执行pip install * 安装依赖的包

部署flask项目
----

1. 上传项目到相应的目录
2. 修改启动配置host和port
