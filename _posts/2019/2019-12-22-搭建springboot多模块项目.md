---
layout:     post
title:      "搭建springboot多模块项目"
subtitle:   ""
date:       2019-12-22 16:50:00
author:     "wangtiegang"
header-img: ""
catalog: true
tags:
    - springboot
    - java
---

### 前言

前段时间将前端项目基于 antd pro 搭建好了，于是就到了搭建后端项目的时候了。在这之前使用 springboot 开发过一个很小的项目，这次准备基于上次的经验，对项目结构进行一些升级。首先，这个项目应该按模块划分，比如登陆，用户，权限，队列等通用的功能应该划分为一块，然后把业务相关的划分为一块，这样以后其他系统就可以方便的复用通用模块，而业务再单独加一个模块就行了。

要实现上面的设想，最好的是将通用模块建立一个单独的项目，然后发布到私有的 maven 仓库，通过 maven 依赖引入其他业务项目，这样一旦通用模块有更新或升级，其他项目可以根据版本控制跟进，降低了耦合，一旦发现bug也不需要多个项目同时修改。

![pj-0](/img/in-post/2019-12/pj-0.png)

设想挺好的，但是现实并不允许，以前没有自己搭过私有maven仓库，也没有弄过发布这些，感觉要把这些东西的来龙去脉摸清楚搭建一遍，需要花不少时间，而时间并不够，所以只能退而求其次，先在同一个项目中实现，将通用和业务当作两个模块集成到项目中，等以后有时间，可以方便的将通用模块抽出来改为maven依赖。

### 搭建springboot多模块项目

搭建多模块项目其实跟 springboot 没有多大联系，主要是使用 maven 实现，而 springboot 替换成其他的也是大同小异的。以前经常使用 maven ，但是对从零搭建也不太熟悉，所以一开始从网上看了很多帖子，摸索了一番之后终于搭建好了。虽然我也无法确认我搭建的是否完全正确，但至少目前使用正常，此处记录下搭建的过程。

#### 搭建项目结构

搭建多模块项目，首先需要一个父项目，这个项目用打包，pom.xml 配置公共的依赖等信息

使用 idea 创建项目，如果没有 Spring Initializr 选项是因为 idea 没有装相关的插件，选择 Maven 也可以。

![pj-1](/img/in-post/2019-12/pj-1.png)

此处取名 ```demo-parent```

![pj-2](/img/in-post/2019-12/pj-2.png)

然后直到结束，此时会生成一个项目，pom.xml 中已经包含 springboot 相关的依赖（如果是选择的 maven 方式，则自己添加相关依赖即可），此时我们先不做任何修改，继续创建模块。

选中 demo-parent 新建一个 module 

![pj-3](/img/in-post/2019-12/pj-3.png)

命名为 demo-core，此 module 就是通用模块

![pj-4](/img/in-post/2019-12/pj-4.png)

一直下一步到结束，然后同样步骤创建一个业务 module ，命名为 demo-business。

此时我们得到了一个如下到项目结构

![pj-5](/img/in-post/2019-12/pj-5.png)

#### 修改 parent 项目

demo-parent 项目不需要运行，首先删除 src 目录，然后修改 pom.xml 文件

```xml
<!-- 新增packaging为pom -->
<packaging>pom</packaging>

<!-- 加载子模块 -->
<modules>
	<module>demo-core</module>
	<module>demo-business</module>
</modules>

<!-- 将打包方式修改为maven打包方式 -->
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.1</version>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
			</configuration>
		</plugin>
	</plugins>
</build>
```

#### 修改 core 项目

core 项目作为通用功能给其他模块依赖，修改 pom.xml

```xml
<!-- 父项目 -->
<parent>
	<groupId>com.example</groupId>
	<artifactId>demo-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
</parent>

<!-- 删除以下，可以从父项目继承 -->
<!--<groupId>com.example</groupId>-->
<!--<version>0.0.1-SNAPSHOT</version>-->

<!-- 设置为jar -->
<packaging>jar</packaging>

<build>
	<!-- 配置 spring boot 的 maven 插件 -->
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

需要注意的是，core 是可以单独启动的，它不应该依赖其他模块的任何功能。

#### 修改 business 模块

business 除了是子模块之外，还依赖于 core 模块，修改 pom.xml 文件

```xml
<!-- 父项目 -->
<parent>
	<groupId>com.example</groupId>
	<artifactId>demo-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
</parent>

<!-- 删除以下，可以从父项目继承 -->
<!--<groupId>com.example</groupId>-->
<!--<version>0.0.1-SNAPSHOT</version>-->

<!-- 设置为jar -->
<packaging>jar</packaging>

<dependencies>
	<!-- 依赖 core 模块 -->
	<dependency>
		<groupId>com.example</groupId>
		<artifactId>demo-core</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>
</dependencies>

<build>
	<!-- 配置 spring boot 的 maven 插件 -->
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

至此，项目结构和模块依赖就完成了。

#### 应用配置

在项目搭建完成之后，还有个问题，某些系统配置需要在每个模块重复配置，这显然是不行的，我希望上层模块可以加载子模块的 yml 配置，这样就不需要多个地方配置了，要实现这个效果，可以使用 Spring 的 profiles 配置

在 business 的 application.yml 中增加配置

```yml
# 加载子配置文件，可以加载到依赖模块的配置文件
spring:
  profiles:
    active: druid, mybatis
```

然后在 core 中新增 ```application-druid.yml```, ```application-mybatis.yml``` 配置文件

![pj-6](/img/in-post/2019-12/pj-6.png)

这样就可以把公共的配置放到 core 模块了，而 business 独有的可以放在自己的模块，不重复配置也不污染 core 模块。

