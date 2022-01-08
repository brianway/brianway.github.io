---
layout: post
title:  SpringMVC学习笔记(14)-SpringMVC校验
date:   2016-03-30 14:28:14 +08:00
category: 入门系列笔记
tags: [SpringMVC]
comments: true
---

本文主要介绍springmvc校验，包括环境准备，校验器配置，pojo张添加校验规则，捕获和显示检验错误信息以及分组校验简单示例。

<!-- more -->

## 校验理解

项目中，通常使用较多是前端的校验，比如页面中js校验。对于安全要求较高点建议在服务端进行校验。

服务端校验：

- 控制层conroller：校验页面请求的参数的合法性。在服务端控制层conroller校验，不区分客户端类型（浏览器、手机客户端、远程调用）
- 业务层service（使用较多）：主要校验关键业务参数，仅限于service接口中使用的参数。
- 持久层dao：一般是不校验的。

## springmvc校验需求

springmvc使用hibernate的校验框架validation(和hibernate没有任何关系)。

校验思路：

页面提交请求的参数，请求到controller方法中，使用validation进行校验。如果校验出错，将错误信息展示到页面。

具体需求：

商品修改，添加校验（校验商品名称长度，生产日期的非空校验），如果校验出错，在商品修改页面显示错误信息。


## 环境准备

我们需要三个jar包：

- hibernate-validator.jar
- jboss-logging.jar
- validation-api.jar

这里我们添加maven依赖

```xml
<!-- hibernate 校验 -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.4.Final</version>
</dependency>
```

查看maven依赖树

```
[INFO] \- org.hibernate:hibernate-validator:jar:5.2.4.Final:compile
[INFO]    +- javax.validation:validation-api:jar:1.1.0.Final:compile
[INFO]    +- org.jboss.logging:jboss-logging:jar:3.2.1.Final:compile
[INFO]    \- com.fasterxml:classmate:jar:1.1.0:compile
```

可以看到，另外两个jar包被`hibernate-validator`依赖，所以不用再额外添加了。


## 配置校验器


- 在springmvc.xml中添加

```xml
<!-- 校验器 -->
<bean id="validator"
      class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <!-- hibernate校验器-->
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator" />
    <!-- 指定校验使用的资源文件，在文件中配置校验错误信息，如果不指定则默认使用classpath下的ValidationMessages.properties -->
    <property name="validationMessageSource" ref="messageSource" />
</bean>
<!-- 校验错误信息配置文件 -->
<bean id="messageSource"
      class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <!-- 资源文件名-->
    <property name="basenames">
        <list>
            <value>classpath:CustomValidationMessages</value>
        </list>
    </property>
    <!-- 资源文件编码格式 -->
    <property name="fileEncodings" value="utf-8" />
    <!-- 对资源文件内容缓存时间，单位秒 -->
    <property name="cacheSeconds" value="120" />
</bean>
```

- 校验器注入到处理器适配器中

```xml
<mvc:annotation-driven conversion-service="conversionService"
                       validator="validator">
</mvc:annotation-driven>
```

- 在CustomValidationMessages.properties配置校验错误信息：

```
#添加校验的错误提示信息
items.name.length.error=请输入1到30个字符的商品名称
items.createtime.isNUll=请输入商品的生产日期
```


## 在pojo中添加校验规则

在ItemsCustom.java中添加校验规则：


```java
public class Items {
    private Integer id;
    //校验名称在1到30字符中间
    //message是提示校验出错显示的信息
    //groups：此校验属于哪个分组，groups可以定义多个分组
    @Size(min=1,max=30,message="{items.name.length.error}")
    private String name;

    private Float price;

    private String pic;

    //非空校验
    @NotNull(message="{items.createtime.isNUll}")
    private Date createtime;
```


## 捕获和显示校验错误信息

```java
@RequestMapping("/editItemsSubmit")
public String editItemsSubmit(
        Model model,
        HttpServletRequest request,
        Integer id,
        @Validated ItemsCustom itemsCustom,
        BindingResult bindingResult)throws Exception {
```

- 在controller中将错误信息传到页面即可

```
//获取校验错误信息
if(bindingResult.hasErrors()){
    // 输出错误信息
    List<ObjectError> allErrors = bindingResult.getAllErrors();

    for (ObjectError objectError :allErrors){
        // 输出错误信息
        System.out.println(objectError.getDefaultMessage());
    }
    // 将错误信息传到页面
    model.addAttribute("allErrors", allErrors);

    //可以直接使用model将提交pojo回显到页面
    model.addAttribute("items", itemsCustom);

    // 出错重新到商品修改页面
    return "items/editItems";
}
```

- 页面显示错误信息：

```jsp
<!-- 显示错误信息 -->
<c:if test="${allErrors!=null }">
	<c:forEach items="${allErrors }" var="error">
		${ error.defaultMessage}<br/>
	</c:forEach>
</c:if>
```

## 分组校验

- 需求：
   - 在pojo中定义校验规则，而pojo是被多个controller所共用，当不同的controller方法对同一个pojo进行校验，但是每个controller方法需要不同的校验
- 解决方法：
   - 定义多个校验分组（其实是一个java接口），分组中定义有哪些规则
   - 每个controller方法使用不同的校验分组




1.校验分组

```java
public interface ValidGroup1 {
	//接口中不需要定义任何方法，仅是对不同的校验规则进行分组
	//此分组只校验商品名称长度

}
```


2.在校验规则中添加分组

```java
//校验名称在1到30字符中间
//message是提示校验出错显示的信息
//groups：此校验属于哪个分组，groups可以定义多个分组
@Size(min=1,max=30,message="{items.name.length.error}",groups = {ValidGroup1.class})
private String name;
```

3.在controller方法使用指定分组的校验

```java
// value={ValidGroup1.class}指定使用ValidGroup1分组的校验
@RequestMapping("/editItemsSubmit")
public String editItemsSubmit(
        Model model,
        HttpServletRequest request,
        Integer id,
        @Validated(value = ValidGroup1.class)ItemsCustom itemsCustom,
        BindingResult bindingResult)throws Exception {
```
