---
layout:     post
title:      "maven+srping+cxf实现WebService"
subtitle:   " \"使用JAX-RS方式实现restful风格服务\""
date:       2019-06-16 18:58:00
author:     "wangtiegang"
header-img: ""
catalog: true
tags:
    - WebService
    - CXF
---

### 添加CXF的maven依赖
进入官网查看文档，发现使用cxf基础功能，只需要添加两个依赖。
```
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-jaxws</artifactId>
    <version>3.1.7</version>
</dependency>
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http</artifactId>
    <version>3.1.7</version>
</dependency>
```

### 发布Webservice服务
* 新建一个接口
  ```java
  @WebService
  public interface IYYYContractService {
    
    String sayHi(String name);
        
  }
  ```
  
* 新建实现类
  ```java
  @WebService(endpointInterface = "com.wtg.webservice.yyy.IYYYContractService", serviceName = "sayHi")
  public class YYYContractServiceImpl implements IYYYContractService {

    @Override
    public String sayHi(String name) {
        System.out.println("sayHi called");
        return "Hello " + name;
    }

  }
  ```

* 在spring context配置文件中添加bean声明
  ```
  <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jaxws="http://cxf.apache.org/jaxws"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd ">

    <jaxws:endpoint id="sayHi" implementor="com.wtg.webservice.yyy.YYYContractServiceImpl" address="/sayHi"/>

  </beans>
  ```

* 最后需要在web.xml中添加cxf拦截器,所有包含/ws/的url将交给cxf处理
  ```
  <servlet>
    <servlet-name>CXFServlet</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>CXFServlet</servlet-name>
    <url-pattern>/ws/*</url-pattern>
  </servlet-mapping>
  ```

* 启动应用，在地址栏输入 http://localhost:9090/ws 可以直接看到一个WebService的汇总信息页面
  ![cxf-1](/img/in-post/2019-06/cxf-1.png)

### 发布restful WebService服务

> 想使用restful风格，官网有三种方式可选，如果选择JAX-RS方式，需要添加如下依赖
```
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-jaxrs</artifactId>
    <version>3.1.7</version>
</dependency>
```

* 新建服务接口
  > 经过测试，接口上加不加注解都行
  ```java
  public interface IYYYContractService {

    String sayHi(String name);

  }
  ```

* 实现类
  ```java
  public class YYYContractServiceImpl implements IYYYContractService {
    /**
    * 方法上加上rs的Path注解就行
    */
    @GET
    @Path("/sayHi")
    @Override
    public String sayHi(@QueryParam("name") String name) {
        System.out.println("sayHi called");
        return "Hello " + name;
    }

  }
  ```
  
* 在spring context配置文件中添加bean声明
  > 此处声明使用rs的标签，跟普通WebService有区别
  ```
  <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jaxrs="http://cxf.apache.org/jaxrs"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd">

    <bean id="yyyContractServiceImpl" class="com.wtg.webservice.yyy.YYYContractServiceImpl"></bean>
    <jaxrs:server address="/sayHiService">
        <jaxrs:serviceBeans>
            <ref bean="yyyContractServiceImpl"></ref>
        </jaxrs:serviceBeans>
    </jaxrs:server>

  </beans>
  ```

* web.xml中的配置cxf拦截器，跟普通的WebService一致
* 启动服务，可以看到服务汇总信息页面
  ![cxf-2](/img/in-post/2019-06/cxf-2.png)

### 问题
> cxf发布服务的代码比较简单，配置多一些，不细心也容易出现问题，一下是碰到的一些问题
* 在服务发布之后，直接在浏览器地址中访问服务器，一直报404
  **这个一般是因为调用服务的时候地址写的不对**。一开始我在spring context文件中配置的address是/sayHi，我服务类中的Path也是/sayHi，启动服务后，页面显示的是 ```http://localhost:9090/ws/sayHi``` ，我误以为是sayHi方法服务地址，实际上这个地方显示的是你配置的address，服务类的地址，没有到具体方法.sayHi方法的地址实际是 ```http://localhost:9090/ws/sayHi/sayHi```。

* 最早我使用cxf最新版本3.3.2，按照官网文档写了demo之后一直不行，报错 Invalid byte tag in constant pool：19
  ![cxf-3](/img/in-post/2019-06/cxf-3.png)
  这个不是因为代码或配置有问题，是因为最新版本使用的某些依赖太新了，比如使用了jdk9的特性，但是我的jdk还是1.8，或者其他依赖，对此不想查具体是哪个导致的，直接将cxf版本降级到3.1.7，问题解决。