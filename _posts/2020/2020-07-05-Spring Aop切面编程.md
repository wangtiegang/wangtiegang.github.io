---
layout:     post
title:      "Spring Aop 切面编程"
subtitle:   ""
date:       2020-07-05 15:08:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - java
---

#### 面向切面编程（AOP）

面向切面编程，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

AOP 常用的一些场景包括

* 日志
* 声明式事物
* 安全
* 缓存

通俗一点来说就是把某个比较通用标准的功能抽象出来放到一个类中（切面）中，然后指定切点（要调用这个切面的地方），这样代码调用到切点时，就会执行切面中的代码了，从而方便的实现在方法执行前，执行后，执行异常等位置调用其他代码，比如记录日志。这样在业务方法中就不需要一个个加记录日志的代码，降低了代码耦合，提升了代码重用性。

#### 面向切面的一些术语

* 通知
  
通知定义了切面是什么以及何时使用，如

  * Before：在方法被调用之前调用通知
  * After：在方法完成之后被调用通知，无论方法执行是否成功
  * AfterReturning：在方法成功执行之后调用通知
  * AfterThrowing：在方法抛出异常后调用通知
  * Around：通知包裹了被通知的方法，在被通知方法调用之前和调用之后执行自定义的行为

* 切面

横切关注点可以被模块化为特殊的类，这些类被称为切面，这些类中定义要在不同类型通知时执行的逻辑

* 连接点（Joinpoint）

* 切点（Poincut）

定义调用切面的地方，常用的比如通过指定注解作为切点，那么凡是使用了该注解的方法都将调用切面方法，还有其他方式指定切点，比如使用正则表达式等等。

#### 使用注解方式实现日志切面

* 定义注解

```java
/**
 * 日志处理注解，作为 AOP切点指示器
 */
@Inherited
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface LogProcess {

    String name();

}
```

* 定义切面类
  
```java
/**
 * 日志处理切面类，必须 @Component，@Aspect
 */
@Aspect
@Component
public class LogAspect {

    /**
     * 使用环绕通知，所有使用了 LogProcess 注解的方法都会调用此切面方法
     * @param pjp 连接点
     * @param logProcess 注解对象
     * @return 方法调用都返回值
     */
    @Around("@annotation(logProcess)")
    public Object inAroundMethod(ProceedingJoinPoint pjp, LogProcess logProcess){

        String name = logProcess.name();

        // 执行方法前
        System.out.println(name + "  执行前");
        Object result = null;
        try{
            // 调用目标方法，获得返回值
            result = pjp.proceed();
            System.out.println(" 执行成功，返回值为： " + result);
        }catch(Exception e){
            System.out.println(" 执行异常");
        }

        return result;

    }

}
```

* 使用方式

```java
/**
* Spring AOP只支持ioc容器管理的Bean，其他的普通java类无法支持aop
*/
@Service
public class UserSerivceimpl implements IUserService{

    /**
     * 使用 LogProcess 注解，调用时会调用切面处理
     * @return
     */
    @LogProcess(name = "getUserName")
    @Override
    public String getUserName(){
        return "wtg";
    }
    
}
```

以上就是切面编程的简单应用，此文省略了切面相关包的引入，同时也忽略了其他方式切点指示器，通知类型的处理。