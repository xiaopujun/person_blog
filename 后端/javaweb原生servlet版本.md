# 创建web工程

此处使用maven创建，以下为步骤

- 使用maven创建，选择webapp骨架
- 补齐web目录结构

![image-20200226132647931](C:\Users\xiaopu\AppData\Roaming\Typora\typora-user-images\image-20200226132647931.png)

- 跟新web.xml版本到最新

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

</web-app>
```

- 配置好本地tomcat服务器

# 使用servlet开发Javaweb工程

编写一个类，继承自HttpServlet，并覆写相关方法（这里的HttpServlet来自于servlet api，需要引入jar包），为了能通过浏览器通过地址访问到我们写的类，需要再web.xml中注册一下我们写的类。

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter writer = resp.getWriter();
        writer.print("hello servlet");
    }
}
```

在web.xml中配置我们的servlet，并形成路径映射关系

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.ctbu.servlet.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

</web-app>
```

如此，通过浏览器就可以完整的访问我们的类并做出逻辑处理。

# ***Servlet***原理

Servlet是由Web服务器调用，web服务器在收到浏览器请求之后，会产生两个对象，也就是请求和相应对象，并把他们交给servlet，但是servlet只是一个接口，因此，需要找到他的实现，此时，通过路径映射就会找到我们自己写的servlet实现，并把两个对象交给我们覆写的方法进行处理。

