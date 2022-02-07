---
layout: post
title:  SpringMVC学习笔记(22)-SpringMVC开发小结
date:   2016-03-30 14:28:22 +08:00
category: 入门系列笔记
tags: [SpringMVC]
comments: true
---

本文对springmvc系列博文进行小结

<!-- more -->

## springmvc框架

![springmvc_核心架构图](/img/blog/springmvc_%E6%A0%B8%E5%BF%83%E6%9E%B6%E6%9E%84%E5%9B%BE.jpg)

- `DispatcherServlet`前端控制器：接收request，进行response
- **`HandlerMapping`处理器映射器**：根据url查找Handler。（可以通过xml配置方式，注解方式）
- **`HandlerAdapter`处理器适配器**：根据特定规则去执行Handler，编写Handler时需要按照HandlerAdapter的要求去编写。
- **`Handler`处理器**（后端控制器）：需要程序员去编写，**常用注解开发方式**。
    - Handler处理器执行后结果是`ModelAndView`，具体开发时`Handler`返回方法值类型包括：`ModelAndView`、`String`（逻辑视图名）、`void`（通过在Handler形参中添加request和response，类似原始 servlet开发方式，注意：可以通过指定response响应的结果类型实现json数据输出）
- `View Resolver`视图解析器：根据逻辑视图名生成真正的视图（在springmvc中使用View对象表示）
- `View`视图：jsp页面，仅是数据展示，没有业务逻辑。



## 注解开发

### 使用注解方式的处理器映射器和适配器

```xml
<!--注解映射器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
<!--注解适配器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
```

在实际开发，使用`<mvc:annotation-driven>`代替上边处理器映射器和适配器配置。

- `@controller`注解必须要加，作用标识类是一个Handler处理器。
- `@requestMapping`注解必须要加，作用：
	- 1、对url和Handler的**方法**进行映射。
	- 2、可以窄化请求映射，设置Handler的根路径，url就是根路径+子路径请求方式
	- 3、可以限制http请求的方法

映射成功后，springmvc框架生成一个Handler对象，对象中只包括 一个映射成功的method。

### 注解开发中参数绑定

将request请求过来的key/value的数据（理解一个串），通过转换（参数绑定的一部分），将key/value串转成形参，将转换后的结果传给形参（整个参数绑定过程）。

springmvc所支持参数绑定：

- 1、默认支持很多类型：`HttpServletRequest`、`response`、`session`、`model/modelMap`(将模型数据填充到request域)
- 2、支持简单数据类型，整型、字符串、日期..等
    - 只要保证request请求的参数名和形参名称一致，自动绑定成功
	- 如果request请求的参数名和形参名称不一致，可以使用`@RequestParam`（指定request请求的参数名），`@RequestParam`加在形参的前边。
- 3、支持pojo类型
    - 只要保证request请求的参数名称和pojo中的属性名一致，自动将request请求的参数设置到pojo的属性中。
- 4、包装类型pojo参数绑定
  - 第一种方法：在形参中添加`HttpServletRequest request`参数，通过request接收查询条件参数。
  - 第二种方法：在形参中让包装类型的pojo接收查询条件参数。
- 5、集合类型参数绑定
  - 数组绑定：方法形参使用数组接收页面请求的多个参数
  - list绑定：使用List接收页面提交的批量数据，通过包装pojo接收，在包装pojo中定义list<pojo>属性
  - map绑定：在包装类中定义Map对象，并添加`get/set`方法，action使用包装对象接收

*注意：形参中即有pojo类型又有简单类型，参数绑定互不影响。*


自定义参数绑定

- 日期类型绑定自定义：

定义的`Converter<源类型，目标类型>`接口实现类，比如：`Converter<String,Date>`,表示：将请求的日期数据串转成java中的日期类型。

*注意：要转换的目标类型一定和接收的pojo中的属性类型一致。*

将定义的Converter实现类注入到处理器适配器中。

```xml
<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>

<!-- conversionService -->
<bean id="conversionService"
	class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
	<!-- 转换器 -->
	<property name="converters">
		<list>
			<bean class="cn.itcast.ssm.controller.converter.CustomDateConverter"/>
		</list>
	</property>
</bean>
```


### springmvc和struts2区别

springmvc面向方法开发的（更接近service接口的开发方式），struts2面向类开发。

springmvc可以单例开发，struts2只能是多例开发。


## 校验

服务端校验：

- 控制层conroller：校验页面请求的参数的合法性。在服务端控制层conroller校验，不区分客户端类型（浏览器、手机客户端、远程调用）
- 业务层service（使用较多）：主要校验关键业务参数，仅限于service接口中使用的参数。
- 持久层dao：一般是不校验的。

一般使用hibernate的校验框架，依赖`hibernate-validator.jar`,`jboss-logging.jar`,`validation-api.jar`这几个jar包

开发步骤

- 在springmvc.xml中添加校验器
- 校验器注入到处理器适配器中
- 在CustomValidationMessages.properties配置校验错误信息
- 在pojo中添加校验规则
- 在控制器中对参数注解`@Validated`来捕获和显示校验错误信息

分组校验

- 定义校验分组
- 在校验规则中添加分组
- 在controller方法使用指定分组的校验

## 数据回显

数据回显有三种方法

- 1.springmvc默认对pojo数据进行回显。
  - pojo数据传入controller方法后，springmvc自动将pojo数据放到request域，key等于pojo类型（首字母小写）
  - 使用`@ModelAttribute`指定pojo回显到页面在request中的key
- 2.`@ModelAttribute`还可以将方法的返回值传到页面
- 3.使用最简单方法使用model，可以不用`@ModelAttribute`


## 异常处理

系统的dao、service、controller出现都通过throws Exception向上抛出，最后由springmvc前端控制器交由异常处理器进行异常处理。

springmvc提供全局异常处理器（一个系统只有一个异常处理器）进行统一异常处理。


全局异常处理器处理思路：

解析出异常类型

- 如果该异常类型是系统自定义的异常，直接取出异常信息，在错误页面展示
- 如果该异常类型不是系统自定义的异常，构造一个自定义的异常类型（信息为“未知错误”）

抛出异常的位置

- 如果与业务功能相关的异常，建议在service中抛出异常。
- 与业务功能没有关系的异常，建议在controller中抛出。

## 上传图片

开发步骤

- 在页面form中提交enctype="multipart/form-data"的数据时，需要springmvc对multipart类型的数据进行解析。
- 在springmvc.xml中配置multipart类型解析器
- 加入上传图片的jar：`commons-fileupload`
- 创建图片虚拟目录存储图片

## json数据交互

两种json数据交互的形式：

- 请求json、输出json，要求请求的是json串，所以在前端页面中需要将请求的内容转成json，不太方便。
- 请求key/value、输出json。此方法比较常用。

需要的依赖：

- `jackson-databind`
- `jackson-mapper-asl`



在注解适配器中加入`messageConverters`

```xml

<!--注解适配器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
	<property name="messageConverters">
	<list>
	<bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"></bean>
	</list>
	</property>
</bean>
```

**注意：如果使用`<mvc:annotation-driven />`则不用定义上边的内容。**

在controller的返回值上加注解`@ResponseBody`来将java对象输出json，返回json格式数据


## RESTful支持

`@RequestMapping(value="/ itemsView/{id}")`：`{×××}`占位符，请求的URL可以是`/viewItems/1`或`/viewItems/2`，通过在方法中使用`@PathVariable`获取{×××}中的×××变量。`@PathVariable`用于将请求URL中的模板变量映射到功能处理方法的参数上。

如果`@RequestMapping`中表示为`/itemsView/{id}`，id和形参名称一致，`@PathVariable`不用指定名称。

同时需要配置前端控制器。若要访问静态资源，还需在springmvc.xml中添加静态资源解析方法,如`<mvc:resources location="/js/" mapping="/js/**"/>`

## 拦截器

### 拦截器定义

定义拦截器，实现`HandlerInterceptor`接口。接口中提供三个方法。可以从名称和参数看出各个接口的顺序和作用

- `public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception`
   - 参数最少，只有三个
   - 进入 Handler方法之前执行
   - 用于身份认证、身份授权。比如身份认证，如果认证通过表示当前用户没有登陆，需要此方法拦截不再向下执行
- `public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception`
  - 多了一个modelAndView参数
  - 进入Handler方法之后，返回modelAndView之前执行
  - 应用场景从modelAndView出发：将公用的模型数据(比如菜单导航)在这里传到视图，也可以在这里统一指定视图
- `public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception`
  - 多了一个Exception的类型的参数
  - 执行Handler完成执行此方法
  - 应用场景：统一异常处理，统一日志处理

### 拦截器的配置

- 针对HandlerMapping配置(一般不推荐)
  - springmvc拦截器针对HandlerMapping进行拦截设置，如果在某个HandlerMapping中配置拦截，经过该HandlerMapping映射成功的handler最终使用该拦截器
- 类似全局的拦截器
  - springmvc配置类似全局的拦截器，springmvc框架将配置的类似全局的拦截器注入到每个HandlerMapping中。


### 拦截器测试及其应用

链式执行测试

- 两个拦截器都放行
  - preHandle方法按顺序执行，postHandle和afterCompletion按拦截器配置的逆向顺序执行
- 拦截器1放行，拦截器2不放行
  - 拦截器1放行，拦截器2 preHandle才会执行。
  - 拦截器2 preHandle不放行，拦截器2 postHandle和afterCompletion不会执行。
  - 只要有一个拦截器不放行，postHandle不会执行。
- 两个拦截器都不放
  - 拦截器1 preHandle不放行，postHandle和afterCompletion不会执行。
  - 拦截器1 preHandle不放行，拦截器2不执行。

应用

- 统一日志处理拦截器，需要该拦截器preHandle一定要放行，且将它放在拦截器链接中第一个位置。
- 登陆认证拦截器，放在拦截器链接中第一个位置。权限校验拦截器，放在登陆认证拦截器之后。（因为登陆通过后才校验权限，当然登录认证拦截器要放在统一日志处理拦截器后面）
