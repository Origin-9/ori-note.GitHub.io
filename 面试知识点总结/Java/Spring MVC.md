#### MVC 模式

![image-20200813115444230](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200813115444230.png)

#### Spring 整合 Spring MVC

- 需要在 web.xml 中配置 DispatcherServlet 。并且需要配置 Spring 监听器ContextLoaderListener

>ContextLoaderListener在启动Tomcat容器的时候，该类的作用就是自动装载ApplicationContext的配置信息，如果没有设置contextConfigLocation的初始参数则会使用默认参数WEB-INF路径下的application.xml文件。

```xml
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener
	</listener-class>
</listener>
<servlet>
	<servlet-name>springmvc</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<!-- 如果不设置init-param标签，则必须在/WEB-INF/下创建xxx-servlet.xml文件，其中xxx是servlet-name中配置的名称。 -->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring/springmvc-servlet.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
```

#### Spring MVC 工作原理

客户端发送请求 -----> 前端控制器接受请求 ----> 交给处理器映射 HandlerMapping 解析请求对应的 Handler（也就是我们平常说的 Controller） ----> HandlerAdapter 会根据 Handler 找到对应的 Handler 处理器，并处理相关逻辑 ----> 返回模型视图 ModelAndView ----> 视图解析器进行解析 ----> 返回视图 View ----> 前端控制器把Model传给View（渲染视图）并返回给用户。

![image-20200813170045193](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200813170045193.png)

#### Spring MVC 重要组件

##### 1. 前端控制器 DispatcherServlet

> **不需要工程师开发,由框架提供（重要）**

接受请求，响应结果，相当于转发器，中央处理器。用户请求到达前端控制器，相当于 MVC 模式作用的 C，由它调用其他组件处理用户的请求，其存在降低了组件之间的耦合性。

##### 2. 处理器映射器 HandlerMapping

根据请求的 url 查找 Handler。SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式。

##### 3. 处理器适配器

按照特定规则执行 Handler即处理器（Controller）。

##### 4. 处理器 Handler

> 需要工程师 开发

编写Handler时按照HandlerAdapter的要求去做。后端控制器，在DispatcherServlet的控制下，Handler对具体的用户请求进行处理。

##### 5. 视图解析器 View resolver

> 由框架提供

View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象。

##### 6. 视图 View

View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）