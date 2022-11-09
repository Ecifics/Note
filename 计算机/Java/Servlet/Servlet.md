# Servlet

[TOC]



## 官网文档
Tomcat API Documentation：https://tomcat.apache.org/tomcat-8.0-doc/servletapi/index.html


## 一、Servlet入门实例-获取请求参数

HTML页面 add.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="add" method="post">
    名称:<input type="text" name="fname"><br>
    价格:<input type="text" name="price"><br>
    库存:<input type="text" name="fcount"><br>
    备注:<input type="text" name="remark"><br>
    <input type="submit" value="添加">
</form>
</body>
</html>
```

> 通过post请求发送，action中的add表示提交表单端url



post请求中的相关内容会封装成HttpServletRequest对象，servlet可以通过这个对象，获取请求相关的内容，Servlet类需要继承HttpServlet类，并且通过HttpServlet的doPost方法来处理post请求，可以通过getParameter方法获取post的请求中的参数

```java
public class AddServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String fname = req.getParameter("fname");
        int price = Integer.parseInt(req.getParameter("price"));
        int fcount = Integer.parseInt(req.getParameter("fcount"));
        String remark = req.getParameter("remark");

        System.out.println("fname = " + fname);
        System.out.println("price = " + price);
        System.out.println("fcount = " + fcount);
        System.out.println("remark = " + remark);
    }
}
```



修改web.xml文件，完成对应url和servlet类的一个映射

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <servlet>
        <servlet-name>AddServlet</servlet-name>
        <servlet-class>com.ecifics.servlet.AddServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>AddServlet</servlet-name>
        <url-pattern>/add</url-pattern>
    </servlet-mapping>
</web-app>
```



示意图

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/servlet/servlet%E5%85%A5%E9%97%A8%E6%A1%88%E4%BE%8B%E7%A4%BA%E6%84%8F%E5%9B%BE.png" align="left" alt="servlet入门案例示意图">





## 二、中文乱码问题

对于post请求，需要设置编码，防止中文乱码，具体操作是在doPost方法的开头（**必须写在开头**）写上如下代码

```java
Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    // 设置字符编码
    req.setCharacterEncoding("UTF-8");
    
    .....
}
```



对于get方式不需要设置编码，如果用get请求发送中文数据，在tomcat8之前版本，需要先将中文字符串转换成ISO-8859编码(Tomcat8之前底层都是ISO-8859编码)字节数组，再将字节数组转换成字符串

```java
Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    req.setCharacterEncoding("UTF-8");
    String fname = req.getParameter("fname");
    byte[] bytes = fname.getBytes("ISO-8859-1");
    fname = new String(bytes, "UTF-8");
}
```



## 三、Servlet继承关系以及其中的方法



### 3.1 继承关系

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/servlet/Servlet%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.png" align="middle" alt="Servlet继承示意图">



所有的自定义Servlet类需要继承HttpServlet（抽象类），而HttpServlet继承至GenericServlet（抽象类），而GenericServlet实现了Servlet、Serializable和ServletConfig接口。



### 3.2 Service方法

其中，Servlet接口规定了如下方法

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

其中Service方法在GenericServlet抽象类中并没有实现，而是交给了HttpServlet实现

```java
public abstract class HttpServlet extends GenericServlet {
    .....

    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getMethod();
        long lastModified;
        if (method.equals("GET")) {
            lastModified = this.getLastModified(req);
            if (lastModified == -1L) {
                this.doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader("If-Modified-Since");
                } catch (IllegalArgumentException var9) {
                    ifModifiedSince = -1L;
                }

                if (ifModifiedSince < lastModified / 1000L * 1000L) {
                    this.maybeSetLastModified(resp, lastModified);
                    this.doGet(req, resp);
                } else {
                    resp.setStatus(304);
                }
            }
        } else if (method.equals("HEAD")) {
            lastModified = this.getLastModified(req);
            this.maybeSetLastModified(resp, lastModified);
            this.doHead(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        } else if (method.equals("PUT")) {
            this.doPut(req, resp);
        } else if (method.equals("DELETE")) {
            this.doDelete(req, resp);
        } else if (method.equals("OPTIONS")) {
            this.doOptions(req, resp);
        } else if (method.equals("TRACE")) {
            this.doTrace(req, resp);
        } else {
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[]{method};
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(501, errMsg);
        }

    }

    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest request;
        HttpServletResponse response;
        try {
            request = (HttpServletRequest)req;
            response = (HttpServletResponse)res;
        } catch (ClassCastException var6) {
            throw new ServletException("non-HTTP request or response");
        }

        this.service(request, response);
    }
}

```

service方法在其中根据请求的类型（Get、Post、Head等等）去调用对应的do方法，例如doGet、doPost等。



### 3.3 为什么用户自定义Servlet类需要重写doGet和doPost方法

在HttpServlet抽象类中，对doGet和doPost方法定义如下

```java
public abstract class HttpServlet extends GenericServlet {
    ......
        
    private static final ResourceBundle lStrings = ResourceBundle.getBundle("javax.servlet.http.LocalStrings");
        
    .....

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_get_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }

    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_post_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }
    
    .....
}
```

> 其中lStrings的定义中，javax.servlet.http.LocalStrings配置文件内容如下
>
> ```properties
> err.cookie_name_is_token=Cookie name \"{0}\" is a reserved token
> err.cookie_name_blank=Cookie name may not be null or zero length
> err.io.nullArray=Null passed for byte array in write method
> err.io.indexOutOfBounds=Invalid offset [{0}] and / or length [{1}] specified for array of size [{2}]
> err.io.short_read=Short Read
> 
> http.method_not_implemented=Method {0} is not implemented by this servlet for this URI
> 
> http.method_get_not_supported=HTTP method GET is not supported by this URL
> http.method_post_not_supported=HTTP method POST is not supported by this URL
> http.method_put_not_supported=HTTP method PUT is not supported by this URL
> http.method_delete_not_supported=Http method DELETE is not supported by this URL
> ```
>
> 

不难看出，如果用户自定义Servlet不重写doPost和doGet方法，会直接调用父类，也就是HttpServlet中对这两个方法的定义，之后会根据http协议的版本号，返回对应的405错误提示，因此需要我们重写doGet和doPost方法



## 四、Servlet生命周期

### 4.1 生命周期概述

Servlet生命周期对应Servlet接口中的核心方法：init()、service()和destroy()

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ....

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    .....

    void destroy();
}
```



默认情况下：

+ 第一次接收请求时，容器内的所有Servlet会使用反射调用构造方法创建实例，之后调用init方法进行初始化操作，最后调用service方法进行服务
+ 第二次请求以及后，调回调用service方法进行服务
+ 当容器（Tomcat）关闭，所有servlet会被销毁



默认情况下的优缺点：

+ 优点：在第一次请求的时候，才会去实例化初始化对象，可以避免系统启动的时候做这些操作，提升了系统的启动时间
+ 缺点：正因为在第一次请求的时候才回去实例化初始化对象，所以响应速度有所降低



测试程序代码如下

```java
public class Demo02Servlet extends HttpServlet {

    public Demo02Servlet() {
        System.out.println("正在实例化");
    }

    @Override
    public void init() throws ServletException {
        System.out.println("正在初始化");
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        System.out.println("正在服务");
    }

    @Override
    public void destroy() {
        System.out.println("正在销毁");
    }
}
```

启动Tomcat，控制台会打印出

```
正在实例化
正在初始化
正在服务
```

之后进行多次请求，控制台打印信息

```
正在服务
正在服务
正在服务
正在服务
正在服务
```

关闭容器（Tomcat），控制台打印

```
正在销毁
```



### 4.2 初始化时机

默认情况下，是在第一次请求是，才进行实例化和初始化操作，但是可以通过修改xml中\<local-on-startup\>属性来设置初始化顺序，数字越小，初始化时机越靠前，最小值为0

```xml
<servlet>
    <servlet-name>Demo02Servlet</servlet-name>
    <servlet-class>com.ecifics.servlet.Demo02Servlet</servlet-class>
    <load-on-startup>0</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>Demo02Servlet</servlet-name>
    <url-pattern>/demo02</url-pattern>
</servlet-mapping>
```

修改在，会在容器启动过程中，就进行实例化和初始化操作，而不是等到第一个请求到来时才进行



## 五、会话技术

### 5.1 Session会话跟踪技术

Http是无状态协议，表示服务器无法分辨两次请求是否为同一个客户端发送的，因此出现了会话跟踪技术。

请求过程如下：

+ 客户端第一次发送请求给服务器，服务器查看请求报文中是否有SessionID，发现没有，于是创建一个Session同时生成了对应的Session ID，通过响应报文发送给了客户端
+ 当客户端再次发送请求给服务器，请求报文中会携带上一次响应报文中的Session ID，服务端获取请求报文后，通过Session ID就可以判别当前请求和之前某个请求是同一个用户发送的



### 5.2 服务器内部转发和客户端重定向

#### 服务端内部转发

服务端内部转发发生在服务端内部，客户端没有感知，地址栏也不会发生改变，通过下面代码进行操作

```java
request.getRequestDispatcher("....").forward(request, response);
```

示例代码，Demo06Servlet接收到请求后，会将该请求转发到Demo07Servlet进行处理

```java
@WebServlet("/demo06")
public class Demo06Servlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("Demo06");
        // 服务端内部转发
        req.getRequestDispatcher("demo07").forward(req, resp);
    }
}
```

```java
@WebServlet("/demo07")
public class Demo07Servlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("Demo07 ....");
    }
}
```

控制台打印信息

```
Demo06
Demo07 ....
```



#### 客户端重定向

客户端重定向发生在客户端发送请求给服务器，服务器上该资源可能已经过期，然后服务器会在响应报文中加上最新资源的访问地址给客户端，客户端收到响应报文后，会访问最新资源对应的服务器。此时会发送两次Http请求，因此地址栏也会发生改变。完成此操作的代码如下

```java
response.sendRedirect("....");
```



示例代码，Demo06Servlet收到对应请求后，会重定向到/demo07交给Demo07Servlet去处理该请求

```java
@WebServlet("/demo06")
public class Demo06Servlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("Demo06");

        resp.sendRedirect("....");
    }
}
```

Demo07Servlet代码同上不变，控制台信息和上面也相同。

