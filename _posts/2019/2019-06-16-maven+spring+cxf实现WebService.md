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

> 最近需要为业务系统提供数据服务，包括json数据和文件，于是先了解一下利用cxf发布WebService服务，对数据传输，安全验证，文件传输进行测试，看看是否有问题。

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

### Json数据支持

CXF支持xml和json格式的数据，默认使用xml，但是官方文档关于这块的资料非常少，查询各种博客后大多也写得没头没尾，最后实际测试出一种可行方案，过程也比较简单。

* cxf将在对象和json之间互转需要提供一个转换类，可以引入jackson的jaxrs包
  
  ```
  <dependency>
    <groupId>com.fasterxml.jackson.jaxrs</groupId>
    <artifactId>jackson-jaxrs-json-provider</artifactId>
    <version>2.9.8</version>
  </dependency>
  ```

* 在spring context中配置JacksonJsonProvider转换类
  
  ```
  <!-- JacksonJsonProvider配置在具体的服务类中 -->
  <jaxrs:server address="/sayHiService">
    <jaxrs:serviceBeans>
        <ref bean="yyyContractServiceImpl"></ref>
    </jaxrs:serviceBeans>
    <jaxrs:providers>
        <bean class="com.fasterxml.jackson.jaxrs.json.JacksonJsonProvider"/>
    </jaxrs:providers>
  </jaxrs:server>
  ```

* 在服务方法中使用@Produces和@Consumes注解方法
  
  ```java
  @POST
  @Path("/json")
  @Produces(MediaType.APPLICATION_JSON) // 指定返回值为json格式，必须
  @Consumes(MediaType.APPLICATION_JSON) // 限制接收参数的类型，类型不对的拒绝接收请求，非必须
  @Override
  public RBean jsonData(PBean pb){

    System.out.println(pb.getName() + " " + pb.getAge() + " " + pb.getDate());

    RBean rb = new RBean();
    rb.setStatus("success");
    rb.setData(Arrays.asList("a","b","c"));

    return rb;
  }
  ```

* 在Postman中测试结果
  
  ![cxf-4](/img/in-post/2019-06/cxf-4.png)

### WebService安全控制
通常对外的业务接口都得进行权限验证，cxf提供了一系列标准的验证，比如xml中配置用户名密码，每次请求Header中都带上用户名和密码进行验证，但是感觉这样不是很好，得给客户端创建用户和密码，每次请求都得带上，而且管理密码也比较烦。在知道客户端服务器ip的前提下，如果通过IP白名单控制权限则好得多，这里我们可以结合cxf Features和Interceptors来实现。

#### CXF Features

Features可以向服务端，客户端，总线（bus）添加扩展功能，比如记录所有请求日志，统一权限验证等等。官方文档看起来Features也是通过Interceptors为基础来实现功能的，我理解是适用场景不同，并且Features可以方便的组合多个Interceptors来实现功能。详细说明可以查看官方文档。

#### CXF Interceptors

Interceptors（拦截器）是CXF内部基本处理单元，调用WebService服务时会创建并调用InterceptorChain（拦截链）。当客户端调用服务时，客户端有一个输出链，服务端有个输入链，服务端返回结果时，服务端有一个输出链，客户端有一个输入链，我们可以自定义拦截器添加到指定链并处理相关逻辑。此处我通过往服务端的输入链添加拦截器实现IP白名单功能。

* 新建一个拦截器继承AbstractPhaseInterceptor<Message>类
  
  > 验证失败之后可以直接throws一个Fault异常，这样cxf会返回一个xml的异常消息给客户端，但是现在一般返回json格式数据，所以可以利用response返回json字符串。

  ```java
  public class TestInvokeInterceptor extends AbstractPhaseInterceptor<Message> {

    public TestInvokeInterceptor(){
        //必须在构造方法中指定拦截的阶段，Phase.RECEIVE表示请求接收到的时候
        //输入链有多个阶段，可以查看官方文档了解
        super(Phase.RECEIVE);
    }

    @Override
    public void handleMessage(Message message) throws Fault {
        HttpServletRequest request = (HttpServletRequest) message.get("HTTP.REQUEST");
        HttpServletResponse response = (HttpServletResponse) message.get("HTTP.RESPONSE");

        // 省略ip白名单验证，可以从配置文件或数据库读取
        if(false == checkIp(request)){
            try {
                // 返回json字符串
                String msg = "{\"errCode\":\"IP-ERR\",\"errMsg\":\"IP address is denied\"}";
                ServletOutputStream out = response.getOutputStream();
                out.write(msg.getBytes("utf-8"));
                out.flush();
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
            // 终止输入链调用，将不再调用接口，直接返回response
            message.getInterceptorChain().abort();
        }
    }

  }
  ```

* 将拦截器配置到服务
  > 方式一：直接在服务类上使用@InInterceptors注解添加，此注解接收一个数组，可以添加多个拦截器。拦截器将只对当前服务类进行拦截。

  ```java
  @InInterceptors(interceptors={"com.wtg.webservice.yyy.interceptor.TestInvokeInterceptor"})
  public class YYYContractServiceImpl implements IYYYContractService {

    @GET
    @Path("/sayHi")
    @Override
    public String sayHi(@QueryParam("name") String name) {
        System.out.println("sayHi called");
        return "Hello " + name;
    }

  }
  ```

  > 方式二：在spring context文件中添加xml配置，将对所有服务类起效

  ```
  <bean id = "testInterceptor" class="com.wtg.webservice.yyy.interceptor.TestInvokeInterceptor"></bean>

  <cxf:bus>
    <cxf:inInterceptors>
      <ref bean="testInterceptor"></ref>
    </cxf:inInterceptors>
  </cxf:bus>
  ```

### 问题
> cxf发布服务的代码比较简单，配置多一些，不细心也容易出现问题，以下是碰到的一些问题

* 在服务发布之后，直接在浏览器地址中访问服务器，一直报404
  **这个一般是因为调用服务的时候地址写的不对**。一开始我在spring context文件中配置的address是/sayHi，我服务类中的Path也是/sayHi，启动服务后，页面显示的是 ```http://localhost:9090/ws/sayHi``` ，我误以为是sayHi方法服务地址，实际上这个地方显示的是你配置的address，服务类的地址，没有到具体方法.sayHi方法的地址实际是 ```http://localhost:9090/ws/sayHi/sayHi```。

* 最早我使用cxf最新版本3.3.2，按照官网文档写了demo之后一直不行，报错 Invalid byte tag in constant pool：19
  ![cxf-3](/img/in-post/2019-06/cxf-3.png)
  这个不是因为代码或配置有问题，是因为最新版本使用的某些依赖太新了，比如使用了jdk9的特性，但是我的jdk还是1.8，或者其他依赖，对此不想查具体是哪个导致的，直接将cxf版本降级到3.1.7，问题解决。

* cxf可以直接将一个类发布为WebService服务，但是官方推荐先定义一个接口，再通过实现类去实现服务。实际测试中发现，如果一个类实现了服务接口，则实现类中的服务方法都必须在接口中存在，否则请求的时候后台会报错，报错信息会直接提示需要在接口中定义方法。不知道是不是没有定义接口的时候，cxf会自动生成接口，而定义了的时候不再生成，则需要自己在接口中定义所有方法。总之注意下就好了。