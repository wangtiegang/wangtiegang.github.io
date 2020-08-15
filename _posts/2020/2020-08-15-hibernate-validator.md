---
layout:     post
title:      "hibernate-validator"
subtitle:   ""
date:       2020-08-15 21:22:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - java
---

平时在接收前端传过来的数据时，需要对数据进行各种基本的校验，比如非空，长度，邮箱，电话格式等，如果每个方法都自己去写校验，就会很烦，而且代码看起来不爽，这个时候可以使用 hibernate-validator 将自己解放出来。

#### 集成 hibernate-validator

在一开始查找资料时，网上很多博客都说 springboot 已经集成了 hibernate-validation，但是我在实际都验证中，却怎么都引入不了相关都注解，再查询资料时才发现，springboot 2.2 之前都内置了 hibernate-validation，2.3 开始，就需要手动引入依赖包。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

#### 相关注解

校验注解其实是根据 JSR-303 标准来定义的验证约束声明和元数据，hibernate-validator 其实是它都一套实现。

* @Null   被注释的元素必须为 null     
* @NotNull    被注释的元素必须不为 null     
* @AssertTrue     被注释的元素必须为 true     
* @AssertFalse    被注释的元素必须为 false     
* @Min(value)     被注释的元素必须是一个数字，其值必须大于等于指定的最小值     
* @Max(value)     被注释的元素必须是一个数字，其值必须小于等于指定的最大值     
* @DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值     
* @DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值     
* @Size(max=, min=)   被注释的元素的大小必须在指定的范围内     
* @Digits (integer, fraction)     被注释的元素必须是一个数字，其值必须在可接受的范围内     
* @Past   被注释的元素必须是一个过去的日期     
* @Future     被注释的元素必须是一个将来的日期     
* @Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式  

Hibernate Validator 附加的 constraint     

* @NotBlank(message =)   验证字符串非null，且长度必须大于0     
* @Email  被注释的元素必须是电子邮箱地址     
* @Length(min=,max=)  被注释的字符串的大小必须在指定的范围内     
* @NotEmpty   被注释的字符串的必须非空     
* @Range(min=,max=,message=)  被注释的元素必须在合适的范围内

#### 使用方式

最简单的使用方式是在 Dto 和 Controller 中直接使用注解

```java
public class Employee {

    @NotNull
    private Long employeeId;

    @NotEmpty(message = "姓名不能为空")
    private String name;

    @Min(value = 0, message = "年龄必须大于0")
    private Integer age;

    @Email(message = "邮件格式不对")
    private String email;

}
```

```java
// 声明使用校验
@Validated
@Controller
public class EmployeeContrllor {

    /**
    * 使用 @Valid 校验数据
    */
    @PostMapping(value = "/employee")
    public void save(@RequestBody @Valid List<Employee> employees) {
        // ...
    }

}
```

也可以直接注入 Validator 进行手动校验

```java
@Service
public class EmployeeServiceImpl {

    @Autowired
    Validator validator;

    public void save(Employee employee) {
        Set<ConstraintViolation<Employee>> constraintViolations = validator.validate(employee);
        for (ConstraintViolation violation :constraintViolations) {
            // 输出错误信息
            System.out.println(violation.getMessage() + violation.getInvalidValue());
        }
    }

}
```

#### 配合 @ControllerAdvice 全局处理校验异常

在 Controller 中使用注解校验，如果不通过会抛出 ConstraintViolationException 异常，但是直接这样返回异常到前端显然是不行到，使用 @ControllerAdvice 注解可以全局拦截返回想要格式

```java
@ControllerAdvice
public class GlobalValidatorExceptionHandler {

    @ExceptionHandler(value = ConstraintViolationException.class)
    public ResponseData<Object> handleMethodArgumentNotValidException(ConstraintViolationException ex) {
        List<Error> errors = new ArrayList<>();
        // 将校验错误转为前端需要的格式
        Set<ConstraintViolation<?>> constraintViolations = ex.getConstraintViolations();
        for (ConstraintViolation<?> constraintViolation : constraintViolations) {
            Error error = new Error();
            error.setDesc(constraintViolation.getMessage());
            errors.add(error);
        }
        return new ResponseData(false, errors);
    }

}
```

以上就是 hibernate-validator 的基本使用，除了这些还可以扩展校验注解来实现项目的特殊校验。