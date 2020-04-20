### Servlet基本概念

#### Servlet是什么

Servlet是一个接口（javax.servlet.Servlet），相当于一个规范，定义实现特定功能的类，这些类在Servlet容器里使用。

#### Servlet接口的方法

- init()
- service()
- destroy()
- getServletConfig()
- getServletInfo()

#### Servlet容器的使用

servlet没有main()方法，是通过servlet容器，比如tomcat，来调用的。

tomcat的webapps目录相当于所有web应用的目录

通过配置tomcat的webapps目录下，你所在web应用目录下的WEB-INF下的web.xml文件，将web request映射到某个servlet来执行。

而你创建的servlet编译生成的class放在你所在web应用目录下的WEB-INF下的classes目录中。

![](https://raw.githubusercontent.com/chenxin2019/ImageHosting/master/Jietu20200414-235330.jpg)



#### Servlet使用方式

servlet-name表示你创建的Servlet类的类名；

servlet-class表示包括package的完整的类名；

在servlet-mapping指定把url映射到哪一个servlet处理。

```xml
<servlet>
	<servlet-name>HelloWorldServlet</servlet-name>
  <servlet-class>HelloWorldServlet</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>HelloWorldServlet</servlet-name>
  <url-pattern>/helloworld</url-pattern>
</servlet-mapping>
```

http://localhost:8080/ch02/helloworld的请求会交由HelloWorldServlet处理



#### Servlet 中service() doGet() doPost()方法关系

https://blog.csdn.net/u010398771/article/details/82758022

service方法是接口中的方法，servlet容器把所有请求发送到该方法，该方法默认行为是转发http请求到doXXX方法中，如果你重载了该方法，默认操作被覆盖，不再进行转发操作！ 

