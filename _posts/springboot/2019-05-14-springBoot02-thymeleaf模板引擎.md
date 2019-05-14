---
layout: post
title:  "springboot(二)：thymeleaf使用详解 "
date:   2019-05-14 13:16:30
categories: Spring Boot
tags: springboot thymeleaf
---

* content
{:toc}
本文是基于spring boot1.5.9，是spring boot系列第二篇，主要讲解thymeleaf模板的使用。


## 1、引入thymeleaf；

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
          	2.1.6
		</dependency>
目前spring-boot 2.x ，只支持thymeleaf2。使用thymeleaf3主程序，需要添加layout2布局支持程序
<properties>
		<thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
		<!-- 布局功能的支持程序  thymeleaf3主程序  layout2以上版本 -->
		<!-- thymeleaf2   layout1-->
		<thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
  </properties>
```



## 2、Thymeleaf使用
Thymeleaf配置类
```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");

	private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
  
```

只要我们把HTML页面放在classpath:/templates/，thymeleaf就能自动渲染；

使用Demo：

1、导入thymeleaf的名称空间

```xml
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

2、使用thymeleaf语法；

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text 将div里面的文本内容设置为 -->
    <div th:text="${hello}">这是显示欢迎信息</div>
</body>
</html>
```
3、java后台程序
```java
1.Controller方式
@Controller
public class HelloController {

    @RequestMapping("hello")
    public String index(Model map) {
        map.addAttribute("hello", "http://blog.didispace.com");
        return "hello";
    }
}
=========================================================================================================================
2.RestController方式
@RestController
public class HelloController {

    @RequestMapping("hello")
    public ModelAndView index(Model map) {
        map.addAttribute("hello", "http://blog.didispace.com");
        return new ModelAndView("hello");
    }
}
说明：
1) RestController = Controller + ResponseBody，如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp/html页面，配置的视图解析器InternalResourceViewResolver不起作用，返回的内容就是Return 里的内容。
2)如果需要返回到指定页面，则需要用 @Controller配合视图解析器InternalResourceViewResolver才行。

```

## 3、thymeleaf语法规则
![](../../assets/springboot/2018-02-04_123955.png)

1）、th:text；改变当前元素里面的文本内容；

​	th：任意html属性；来替换原生属性的值


2）、基础语法
变量表达式  ${}
使用方法：直接使用th:xx = "${}" 获取对象属性 。例如：
```html
<form id="userForm">
    <input id="id" name="id" th:value="${user.id}"/>
    <input id="username" name="username" th:value="${user.username}"/>
    <input id="password" name="password" th:value="${user.password}"/>
</form>
<div th:text="hello"></div>

<div th:text="${user.username}"></div>
```
选择变量表达式 *{}
使用方法：首先通过th:object 获取对象，然后使用th:xx = "*{}"获取对象属性。

这种简写风格极为清爽，推荐大家在实际项目中使用。 例如：
```html
<form id="userForm" th:object="${user}">
    <input id="id" name="id" th:value="*{id}"/>
    <input id="username" name="username" th:value="*{username}"/>
    <input id="password" name="password" th:value="*{password}"/>
</form>
```
链接表达式 @{}
```properties
Link URL Expressions: @{...}：定义URL；
```
使用方法：通过链接表达式@{}直接拿到应用路径，然后拼接静态资源路径。例如：
```html
<script th:src="@{/webjars/jquery/jquery.js}"></script>
<link th:href="@{/webjars/bootstrap/css/bootstrap.css}" rel="stylesheet" type="text/css">
```
片段表达式 ~{}
```properties
 Fragment Expressions: ~{...}：片段引用表达式
```
片段表达式是Thymeleaf的特色之一，细粒度可以达到标签级别，这是JSP无法做到的。
```html
片段表达式拥有三种语法：
 1、~{ viewName } 表示引入完整页面  
 2. ~{ viewName ::selector} 表示在指定页面寻找片段 其中selector可为片段名、jquery选择器等
 3. ~{ ::selector} 表示在当前页寻找
 
 
```
使用方法：首先通过th:fragment定制片段 ，然后通过th:replace 填写片段路径和片段名。例如：
```html
<!-- /views/common/head.html-->
<head th:fragment="static">
        <script th:src="@{/webjars/jquery/3.3.1/jquery.js}"></script>
</head>

<!-- /views/your.html -->
<div th:replace="~{common/head::static}"></div>
-------------------------------------------------------------------------------
<div th:replace="common/head::static"></div>
```
消息表达式
```properties
Message Expressions: #{...}：获取国际化内容
```
即通常的国际化属性：#{msg} 用于获取国际化语言翻译值。例如：
```html
 <title th:text="#{user.title}"></title>
```
其它表达式
在基础语法中，默认支持字符串连接、数学运算、布尔逻辑和三目运算等。
```properties
Literals（字面量）
      Text literals: 'one text' , 'Another one!' ,…
      Number literals: 0 , 34 , 3.0 , 12.3 ,…
      Boolean literals: true , false
      Null literal: null
      Literal tokens: one , sometext , main ,…
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
```
例如：
```html
<input name="name" th:value="${'I am '+(user.name!=null?user.name:'NoBody')}"/>
```

3）、内置对象
```properties
七大基础对象：
    #ctx : the context object. //上下文对象，可用于获取其它内置对象。
    #vars: the context variables. //上下文变量。
    #locale : the context locale. //上下文区域设置。
    #request : (only in Web Contexts) the HttpServletRequest object. //HttpServletRequest对象
    #response : (only in Web Contexts) the HttpServletResponse object. //HttpServletResponse对象
    #session : (only in Web Contexts) the HttpSession object.  //HttpSession对象
    #servletContext : (only in Web Contexts) the ServletContext object. //ServletContext对象

常用的工具类：
    #strings：字符串工具类
    #lists：List 工具类
    #arrays：数组工具类
    #sets：Set 工具类
    #maps：常用Map方法。
    #objects：一般对象类，通常用来判断非空
    #bools：常用的布尔方法。
    #execInfo：获取页面模板的处理信息。
    #messages：在变量表达式中获取外部消息的方法，与使用＃{...}语法获取的方法相同。
    #uris：转义部分URL / URI的方法。
    #conversions：用于执行已配置的转换服务的方法。
    #dates：时间操作和时间格式化等。
    #calendars：用于更复杂时间的格式化。
    #numbers：格式化数字对象的方法。
    #aggregates：在数组或集合上创建聚合的方法。
    #ids：处理可能重复的id属性的方法。

```

4、迭代循环
想要遍历List集合很简单，配合th:each 即可快速完成迭代。例如遍历用户列表：
```html
<div th:each="user:${userList}">
    账号：<input th:value="${user.username}"/>
    密码：<input th:value="${user.password}"/>
</div>
```
在集合的迭代过程还可以获取状态变量，只需在变量后面指定状态变量名即可，状态变量可用于获取集合的下标/序号、总数、是否为单数/偶数行、是否为第一个/最后一个。例如：
```html
<div th:each="user,stat:${userList}" th:class="${stat.even}?'even':'odd'">
    下标：<input th:value="${stat.index}"/>
    序号：<input th:value="${stat.count}"/>
    账号：<input th:value="${user.username}"/>
    密码：<input th:value="${user.password}"/>
</div>
```
如果缺省状态变量名，则迭代器会默认帮我们生成以变量名开头的状态变量 xxStat， 例如：
```html
<div th:each="user:${userList}" th:class="${userStat.even}?'even':'odd'">
    下标：<input th:value="${userStat.index}"/>
    序号：<input th:value="${userStat.count}"/>
    账号：<input th:value="${user.username}"/>
    密码：<input th:value="${user.password}"/>
</div>
```

5、条件判断
条件判断通常用于动态页面的初始化，例如：
```html
<div th:if="${userList}">
    <div>的确存在..</div>
</div>
```
6、日期格式化
使用默认的日期格式(toString方法) 并不是我们预期的格式：Mon Dec 03 23:16:50 CST 2018
此时可以通过时间工具类#dates来对日期进行格式化：2018-12-03 23:16:50
```html
<input type="text" th:value="${#dates.format(user.createTime,'yyyy-MM-dd HH:mm:ss')}"/>
```

7.内联写法
（1）为什么要使用内联写法？·答：因为 JS无法获取服务端返回的变量。

（2）如何使用内联表达式？答：标准格式为：[[${xx}]] ，可以读取服务端变量，也可以调用内置对象的方法。例如获取用户变量和应用路径：
```javascript
<script th:inline="javascript">
        var user = [[${user}]];
        var APP_PATH = [[${#request.getContextPath()}]];
        var LANG_COUNTRY = [[${#locale.getLanguage()+'_'+#locale.getCountry()}]];
</script>
```
（3）标签引入的JS里面能使用内联表达式吗？答：不能！内联表达式仅在页面生效，因为Thymeleaf只负责解析一级视图，不能识别外部标签JS里面的表达式。

## 4、thymeleaf在spring boot中的常见应用

1）、禁用模板引擎的缓存

```
# 禁用thymeleaf缓存
spring.thymeleaf.cache=false 
```
2）、登陆页面错误消息的显示
   
   ```html
   <p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
   ```
3）、thymeleaf公共页面元素抽取
   
   ```html
   1、抽取公共片段
   <div th:fragment="copy">
   &copy; 2011 The Good Thymes Virtual Grocery
   </div>
   
   2、引入公共片段
   <div th:insert="~{footer :: copy}"></div>
   ~{templatename::selector}：模板名::选择器
   ~{templatename::fragmentname}:模板名::片段名
   
   3、默认效果：
   insert的公共片段在div标签中
   如果使用th:insert等属性进行引入，可以不用写~{}：
   行内写法可以加上：[[~{}]];[(~{})]；
   ```
   
   三种引入公共片段的th属性：
   
   **th:insert**：将公共片段整个插入到声明引入的元素中
   
   **th:replace**：将声明引入的元素替换为公共片段
   
   **th:include**：将被引入的片段的内容包含进这个标签中
   
  **使用th:insert等属性进行引入时，开头请勿添加斜杠，避免部署运行的时候出现路径报错。**
 **（因为默认拼接的路径为spring.thymeleaf.prefix = classpath:/templates/）**  
   
   ```html
   <footer th:fragment="copy">
   &copy; 2011 The Good Thymes Virtual Grocery
   </footer>
   
   引入方式
   <div th:insert="footer :: copy"></div>
   <div th:replace="footer :: copy"></div>
   <div th:include="footer :: copy"></div>
   
   效果
   <div>
       <footer>
       &copy; 2011 The Good Thymes Virtual Grocery
       </footer>
   </div>
   
   <footer>
   &copy; 2011 The Good Thymes Virtual Grocery
   </footer>
   
   <div>
   &copy; 2011 The Good Thymes Virtual Grocery
   </div>
   ```

   引入片段的时候传入参数： 
   
   ```html
   
   <nav class="col-md-2 d-none d-md-block bg-light sidebar" id="sidebar">
       <div class="sidebar-sticky">
           <ul class="nav flex-column">
               <li class="nav-item">
                   <a class="nav-link active"
                      th:class="${activeUri=='main.html'?'nav-link active':'nav-link'}"
                      href="#" th:href="@{/main.html}">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home">
                           <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                           <polyline points="9 22 9 12 15 12 15 22"></polyline>
                       </svg>
                       Dashboard <span class="sr-only">(current)</span>
                   </a>
               </li>
   
   <!--引入侧边栏;传入参数-->
   <div th:replace="commons/bar::#sidebar(activeUri='emps')"></div>
   ```
   
 参考资料：![SpringBoot Thymeleaf使用教程（实用版）](https://www.jianshu.com/p/908b48b10702)
 