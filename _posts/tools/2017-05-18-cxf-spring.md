---
layout: post
title: "CXF系列之和spring的结合"
date: 2017-05-18 09:55:18 +0800
categories: 工具学习
tag: cxf,spring
---

* content
{:toc}



　　CXF和spring结合学习以及发布
    <!-- more -->
    
##第一步配置MAVEN依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>1.0</groupId>
    <artifactId>1.0</artifactId>
    <version>1.0-SNAPSHOT</version>
<dependencies>
    <!--web-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>3.0-alpha-1</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.2.1-b03</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
    <dependency>
        <groupId>javax.ws.rs</groupId>
        <artifactId>javax.ws.rs-api</artifactId>
        <version>2.0.1</version>
    </dependency>
    <dependency>
        <groupId>javax.websocket</groupId>
        <artifactId>javax.websocket-api</artifactId>
        <version>1.0</version>
    </dependency>
    <dependency>
        <groupId>javax.annotation</groupId>
        <artifactId>javax.annotation-api</artifactId>
        <version>1.2</version>
    </dependency>
    <dependency>
        <groupId>javax.transaction</groupId>
        <artifactId>javax.transaction-api</artifactId>
        <version>1.2</version>
    </dependency>
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>7.0</version>
    </dependency>
    <dependency>
        <groupId>com.sun.xml.ws</groupId>
        <artifactId>jaxws-ri</artifactId>
        <version>2.2.10</version>
    </dependency>
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-rt-frontend-jaxws</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-rt-transports-http-jetty</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.2.3.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.2.3.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>4.2.3.RELEASE</version>
    </dependency>
</dependencies>
</project>
```
##第二步：编写WS接口及其实现

```
package demo.ws.ri;

import javax.jws.WebService;

/**
 * Created by lizhaoz on 2016/4/23.
 */
@WebService
public interface HelloService {
    String say(String name);
}
```
实现类

```
package demo.ws.ri;

import org.springframework.stereotype.Component;

import javax.jws.WebService;
import java.util.Set;

/**
 * Created by lizhaoz on 2016/4/23.
 */
@WebService
@Component
public class HelloServiceImpl {
    public String say(String name){
        return "hello"+name;
    }
}
```
需要在实现类上添加Spring的@Component注解，这样才能被IOC容器扫描到，认为它是一个SpringBean,可以根据BeanID来获取Bean实例。
##第三步：配置web.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.susn.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
    <!--Spring-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>cxf</servlet-name>
        <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>cxf</servlet-name>
        <url-pattern>/ws/*</url-pattern>
    </servlet-mapping>
</web-app>
```
所有带ws前缀的请求交给CXFServlet进行处理，也就是处理WS请求。目前主要使用了SpringIOC的特性，利用ContextLoaderListener加载Spring的配置文件。
##第四步：配置Spring
配置spring.xml

```
<?xml version="1.0" encoding="utf-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
	http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd"
       >
    <context:component-scan base-package="demo.ws.ri"/>
    <import resource="spring-cxf.xml"/>
    </beans>
```
##第五步：配置CXF
在配置CXF中有两种配置方式

第一种配置spring-cxf.xml:

```
<?xml version="1.0" encoding="utf-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"

        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

        xmlns:jaxws="http://cxf.apache.org/jaxws"

        xsi:schemaLocation="

        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd

        http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
    <jaxws:server id="helloService" address="/soap/hello">
        <jaxws:serviceBean>
            <ref bean="helloServiceImpl"/>
        </jaxws:serviceBean>
    </jaxws:server>
</beans>
```
通过CSF提供的Spring命名空间发布WS，其中最重要的是addresshe jaxws:serviceBean配置的Bean。
可见，在spring中集成CXF比想象中更简单，还有一种更简单，直接使用CXF提供的endpoint方式,配置如下

```
<?xml version="1.0" encoding="utf-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"

       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

       xmlns:jaxws="http://cxf.apache.org/jaxws"

       xsi:schemaLocation="

        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd

        http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
<!--    <jaxws:server id="helloService" address="/soap/hello">
        <jaxws:serviceBean>
            <ref bean="helloServiceImpl"/>
        </jaxws:serviceBean>
    </jaxws:server>-->
<jaxws:endpoint id="helloService" implementor="#helloServiceImpl" address="/soap/hello"/>
</beans>
```
这里的implementor是CXF特有的简写方式，并非是Spring的规范，通过Spring获取BeanID。这种方式也是我们推崇的方式。
##第六步：部署到tomcat。
输入localhost:8080/cxf/ws即可。
