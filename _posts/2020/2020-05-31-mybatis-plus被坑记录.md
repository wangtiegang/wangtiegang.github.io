---
layout:     post
title:      "mybatis-plus自动填充被坑记录"
subtitle:   ""
date:       2020-05-31 15:45:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - java
---

在新搭建的后端应用中，我使用了 mybatis-plus 框架作为扩展，减少 sql 的书写，提升开发效率，其中有个功能很好用，就是自动填充。在项目中，我们的后台表一般会具有一些标准字段，作为强制建表规范，比如

* object_version_number 乐观锁版本
* create_by 创建者
* creation_date 创建时间
* last_updated_by 最后更新者
* last_update_date 最后更新时间
  
这些字段我们会放到 BaseDTO 中，作为其他 DTO 的超类，这样就不用重复定义了，但是有个问题，在每次修改数据时，都需要给这几个字段赋值，这显然就有点繁琐了，如果能系统自动读取相关值并填充就能省略很多代码，mybatis plus 的自动填充正是一个这样的功能。

#### 自动填充的使用

在使用前，不得不吐槽一下这个功能的 api 改动太频繁了，变化了两三次，文档写的还不太全面，导致一不小心就会被坑。

* 在需要被填充的字段上使用注解，声明什么时候要被填充

```java
public class BaseDTO implements Serializable {

    /**
    * FieldFill.INSERT 只在插入时填充
    * FieldFill.INSERT_UPDATE 插入和更新时都填充
    */

    @Version
    @TableField(fill = FieldFill.INSERT)
    private Integer objectVersionNumber;

    @TableField(fill = FieldFill.INSERT)
    private Long createdBy;

    @TableField(fill = FieldFill.INSERT)
    private Date creationDate;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Long lastUpdatedBy;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private lastUpdateDate;

    /** 省略 getter setter */
}
```

* 自定义实现类 MyMetaObjectHandler

```java
@component
public class MyMetaObjectHandler implements MyMetaObjectHandler {

    /**
    * 插入时填充方法
    */
    @Override
    public void insertFill(MetaObject metaObject) {
        // userId 在实际项目中可以从权限框架中全局获取
        userId = '10001';

        this.strictInsertFill(metaObject, "objectVersionNumber", Integer.class, 1);
        this.strictInsertFill(metaObject, "createdBy", Long.class, userId);
        this.strictInsertFill(metaObject, "creationDate", Date.class, new Date());
        this.strictInsertFill(metaObject, "lastUpdatedBy", Long.class, userId);
        this.strictInsertFill(metaObject, "lastUpdateDate", Date.class, new Date());
    }

    /**
    * 更新时填充方法
    */
    @Override
    public void updateFill(MetaObject metaObject) {
        // userId 在实际项目中可以从权限框架中全局获取
        userId = '10001';

        this.strictUpdateFill(metaObject, "lastUpdatedBy", Long.class, userId);
        this.strictUpdateFill(metaObject, "lastUpdateDate", Date.class, new Date());
    }

}
```

#### 自动填充忽略非空字段的坑

自动填充功能至此就完成了，感觉非常实用，然后用着用着发现不对劲了，数据在更新后，最后更新时间和最后更新人不会被更新，明明插入填充好好的啊，更新填充也是按照文档来的，不可能一个行一个不行啊，调试发现 metaObject 的值确实被更新了，怎么到了数据库就还是旧的呢？接着打印 sql，发现 sql 中的值并没有被更新，last* 两个字段都被忽略了，这就想不通了，没办法只好进入 issuc 搜索，于是发现作者说当被填充的字段有值时会忽略填充。。。这么重要的点，文档中竟然没写，而且感觉有点不合常理。不过知道了原因，要解决也非常简单，直接在 MyMetaObjectHandler 中重写 MyMetaObjectHandler 接口中的 default 实现。

```java
@component
public class MyMetaObjectHandler implements MyMetaObjectHandler {
    
    /** 省略 insertFill updateFill */

    @Override
    public MetaObjectHandler  strictFillStrategy(MetaObject metaObject, String fieldName, Supplier<Object> fieldVal){
        Object obj = fieldVal.get();
        if (Objects.nonNull(obj)){
            metaObject.setValue(fieldName, obj);
        }
        return this;
    }
}
```

#### mybatis plus 忽略 null 更新的坑

在这还必须说一个坑，那就是 mybatis plus 默认忽略 null 的更新，就说是你不能使用标准的 api 方法把字段更新为空，除非使用 wraaper 包装要更新的 dto ，这个也没什么，但是问题在于文档写的很模糊，容易误会文档的意思，导致没有注意到这个问题。

这个可以在配置文件中声明：

```yml
global-config:
  db-config:
    # ignored: 忽略非空判断，所有字段都可以被更新为 null
    # not_null: 更新为 null 时忽略更新
    # not_empty: 字符串不能被更新为 null 或 空，其他类型不能被更新为 null
    update-strategy: ignored
```

对应这个更新策略，见过一些 mysql 数据库要求字段不能为 null 的情况，但是了解不多。在我常用的 oracle 数据库中，是没有这种限制的，所以默认不能更新为 null 对 oracle 来说不太友好，关键是文档写的不够清楚，容易造成bug。