---
layout:     post
title:      "mybatis-plus扩展通用方法"
subtitle:   ""
date:       2020-06-07 15:48:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - java
---

之所以使用 mybatis-plus 框架，就是因为提供了很多标准的增删改查方法，可以避免手写 xml，提高开发效率，但是有时候官方提供的方法并不够，我们想定义一些切合自己业务的通用方法，这个时候就需要自己扩展了，好在官方支持自定义扩展，唯一的缺点就是官方文档写得太简单了，所以结合网上资料，自己实现了一遍。以下以自定义一个查询方法为例。

* 定义一个脚本枚举类，定义方法的脚本

  这个不是必须的，但是源码中是这样写的，我们可以一样定义一个

  ```java
  public enum CustomSqlMethod {

      /**
      * 如果是动态sql，需要在脚本中加<script>标签，如："<script>SELECT %s FROM %s WHERE %s=#{%s} %s</script>"
      */
      SELECT_BY_ID_CUSTOM("selectByIdCustom","自定义查询方法","SELECT %s FROM %s WHERE %s=#{%s} %s");

      private final String method;
      private final String desc;
      private final String sql;

      private CustomSqlMethod(String method, String desc, String sql){
          this.method = method;
          this.desc = desc;
          this.sql = sql;
      }

      public String getMethod(){
          return this.method;
      }

      public String getDesc(){
          return this.desc;
      }

    public String getSql(){
          return this.sql;
      }

  }
  ```

* 继承 ```AbstractMethod``` 定义 method

  ```java
  /**
  * 假设我们自定义的方法是在查询的表格后面统一加上 _custom 后缀
  * 这个方法是最关键的地方，会在应用启动时自动生成 xml 
  */
  public class SelectByIdCustom extends AbstractMethod {

    public SelectByIdCustom(){

    }

    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
        // 自定义枚举中的脚本
        SqlMethod sqlMethod = CustomSqlMethod.SELECT_BY_ID_CUSTOM;
        // 构造 SqlSource，此处可以替换掉表名
        SqlSource sqlSource = new RawSqlSource(configuration, String.format(sqlMethod.getSql(),
            sqlSelectColumns(tableInfo, false),
            tableInfo.getTableName() + "_custom", tableInfo.getKeyColumn(), tableInfo.getKeyProperty(),
            tableInfo.getLogicDeleteSql(true, true)), Object.class);
        return this.addSelectMappedStatementForTable(mapperClass, getMethod(sqlMethod), sqlSource, tableInfo);
    }
  }
  ```

* 将自定义通用方法添加到注入器

  ```java
  /**
  * 该类需要添加到 spring
  */
  @Component
  public class CustomSqlInjector extends DefaultSqlInjector {

      @Override
      public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
          // 需要先获取父类中的通用方法集
          List<AbstractMethod> methods = super.getMethodList(mapperClass);

          // 添加自定义方法
          methods.add(new SelectByIdCustom());
      }

  }
  ```

* 定义 CustomBaseMapper 接口，所有实现此接口的 mapper 都将具有自定义方法

  ```java
  /**
  * 继承 BaseMapper 基础方法，添加自定义方法
  */
  public interface CustomBaseMapper<T> extends BaseMapper<T> {

    /*
    * 方法名必须跟枚举中到 method 属性一致
    */
    T selectByIdCustom(Serializable id);

  }
  ```

* 定义 ICustomService 接口

  ```java
  /**
  * 实现此接口到服务类将具有自定义方法
  */
  public interface ICustomService<T> extends IService<T> {

      // 默认实现，直接调用 mapper
      default T getByIdCustom(Serializable id){
          return this.getCustomBaseMapper().selectByIdCustom(id);
      }

      // 获取注入的 CustomBaseMapper，可以直接返回 IService 中的 baseMapper
      CustomBaseMapper<T> getCustomBaseMapper();
  }
  ```

#### 总结

以上就是扩展自定义方法的过程，可以看到其实非常简单，原理也是利用应用启动时为每一个实现了 CustomBaseMapper 的对象自动生成 xml 脚本，利用 mybatis 的 api 注入通用的 statement，后续用户就可以直接调用，跟自己手写 xml 没有本质区别。

值得一提的是，如果你想自定义方法的同时，隐藏或不推荐使用原生的通用方法，比如你希望提供一个简化版本的 mybatis plus，那么不应该像上述例子中一样使用继承，而是应该使用组合，让自己的 ICustService 只暴露想要的方法，否则无法隐藏原生的 api。本来我想实现的就是这样的效果，但是做完了发现无法使用 ```@Deprecated``` 去提示父类的方法，最后查询资料才看到大神的答案幡然醒悟。