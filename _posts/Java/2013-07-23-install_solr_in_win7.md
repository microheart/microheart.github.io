---
layout: post
title: Windows7下安装Solr
category: Java
tags: [Solr,安装]
keywords: 
description: 
---

##Solr简介
**[Solr](http://lucene.apache.org/solr/)**是一个开放源码、高性能、基于Lucene的企业搜索平台(enterprise search platform)，由Apache软件基金会所研发。

##安装Java
从[Oracle官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)上下载当前最新版本**jdk-7u25-windows-x64.exe**。

安装，并配置JAVA_HOME,path和classpath等环境变量。

##安装Tomcat
1 **下载**

从[Apache Tomcat](http://tomcat.apache.org/download-70.cgi)页面下载**apache-tomcat-7.0.42-windows-x64.zip**，解压至D:\Program Files（可以为任意目录）。

2 **修改server.xml**

    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" 
     redirectPort="8443" URIEncoding="UTF-8">
添加URIEncoding="UTF-8"，以便支持中文

3 **测试**

双击**startup.bat**(bin目录下)启动tomcat，输入**`http://localhost:8080/`**

**注**: 启动Tomcat需要配置JAVA_HOME

##安装Solr
1 **解压**

从[Apache Solr官网](http://lucene.apache.org/solr/)页面下载**solr-4.3.1.zip**(当前最新版本)，解压到任意目录，我的目录为D:\Program Files。

2 **拷贝**

从solr-4.3.1\example\webapps目录下拷贝**solr.war**到tomcat目录下的webapp目录下，这时Tomcat会自动解压(若已启动Tomcat)solr.war,并报错，没事。

3 **配置**

创建配置文件Tomcat home/conf/Catalina/localhost/solr.xml 

    <Context docBase="D:/Program Files/apache-tomcat-7.0.42/webapps/solr" debug="0" crossContext="true" >
       <Environment name="solr/home" type="java.lang.String" value="d:/data/solr" override="true" />
    </Context>
**docBase**设置为solr的绝对路径,

**solr/home**的值设置为存放索引数据等的根路径(本例放在d:/data)，把solr-4.3.1\example的solr文件夹拷贝到d:/data。

**jar包拷贝**

将solr-4.3.1\example\lib\ext目录下的jar包拷贝到tomcat home\webapps\solr\WEB-INF\lib下。

将solr-4.3.1\example\resources的log4j.properties同样拷贝到tomcat home\webapps\solr\WEB-INF\lib下。

**注**：tomcat报以下错误，是因为没有完成jar包的拷贝。

    org.apache.catalina.core.StandardContext startInternal SEVERE: Error filterStart

4 **测试**

打开`http://localhost:8080/solr/`

![solr home](/public/upload/java/solr_home.PNG)