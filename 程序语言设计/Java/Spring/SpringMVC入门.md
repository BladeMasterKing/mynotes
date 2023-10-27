# SpringMVC入门

## MVC模式简介

### 1.1 Web开发的响应模型

![响应模型](media/16885383301166/%E5%93%8D%E5%BA%94%E6%A8%A1%E5%9E%8B.jpeg)

在Web世界里，具体步骤如下：
1. Web浏览器（如IE）发起请求，如访问http://sishuok.com
2. Web服务器（如Tomcat）接收请求，处理请求（比如用户新增，则将把用户保存一下），最后产生响应（一般为html）。
3. web服务器处理完成后，返回内容给web客户端（一般就是我们的浏览器），客户端对接收的内容进行处理（如web浏览器将会对接收到的html内容进行渲染以展示给客户）。
 
因此，在Web世界里：
都是Web客户端发起请求，Web服务器接收、处理并产生响应。
一般Web服务器是不能主动通知Web客户端更新内容。虽然现在有些技术如服务器推（如Comet）、还有现在的HTML5 websocket可以实现Web服务器主动通知Web客户端。
到此我们了解了在web开发时的请求/响应模型，接下来我们看一下标准的MVC模型是什么。
 
### 1.2 标准MVC模型概述

**MVC模型**：是一种架构型的模式，本身不引入新功能，只是帮助我们将开发的结构组织的更加合理，使展示与模型分离、流程控制逻辑、业务逻辑调用与展示逻辑分离
![MVC模型](media/16885383301166/MVC%E6%A8%A1%E5%9E%8B.jpeg)

**MVC（Model-View-Controller）** 三元组的概念：
* **Model（模型）**：数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据） 和 服务层（行为）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。
* **View（视图）**：负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。
* **Controller（控制器）**：接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。 也就是说控制器做了个调度员的工作。

在标准的MVC中模型能主动推数据给视图进行更新（观察者设计模式，在模型上注册视图，当模型更新时自动更新视图），但在Web开发中模型是无法主动推给视图（无法主动更新用户界面），因为在Web开发是请求-响应模型。
在 `Web` 里 `MVC` 区别于标准的 `MVC`，在 `Web MVC` 模式下，模型无法主动推数据给视图，如果用户想要视图更新，需要再发送一次请求（即请求-响应模型）。

### 1.3 Web端开发的发展历程

![Web端发展历程](media/16885383301166/Web%E7%AB%AF%E5%8F%91%E5%B1%95%E5%8E%86%E7%A8%8B.jpeg)

#### CGI
`Common Gateway Interface`公共网关接口，，一种在web服务端使用的脚本技术，使用`C`或`Perl`语言编写，用于接收`web`用户请求并处理，最后动态产生响应给用户，但每次请求将产生一个进程，重量级。

#### Servlet
一种`JavaEE``web`组件技术，是一种在服务器端执行的`web`组件，用于接收`web`用户请求并处理，最后动态产生响应给用户。但每次请求只产生一个线程（而且有线程池），轻量级。而且能利用许多`JavaEE`技术（如`JDBC`等）。本质就是在`java`代码里面 输出`html`流。但表现逻辑、控制逻辑、业务逻辑调用混杂。
例如：
```java
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet (HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }
    
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String submitFlag = req.getParameter("submitFlag");
        if("toLogin".equals(submit)){
            toLogin(req, resp);return;
        } else if("login".equals(submitFlag)){
            login(req, resp);return;
        }
    }
    
    private void toLogin(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.setContentType("text/html");
        String loginPath = req.getContextPath() + "/servletLogin";
        PrintWriter writer = resp.getWriter();
        writer.write("<form action='"+loginPath+"' method='post'>");
        writer.write("<input type='text' name='submitFlag' value='login'/>");
        writer.write("username:<input type='text' name='username'/>");
        writer.write("password:<input type='password' name='password'/>");
        writer.write("<input type='submit' value='login'/>");
        writer.write("</form>");
    }
    
    private void login(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        String username = req.getParamater("username");
        String password = req.getParameter("password");
        UserBean user = new UserBean();
        user.setUsername(username);
        user.setPassword(password);
        if(user.login()){
            resp.getWriter().write("login success");
        } else {
            resp.getWriter().write("login fail");
        }
    }
}
```
这种做法是绝对不可取的，控制逻辑、表现代码、业务逻辑对象调用混杂在一起，最大的问题是直接在Java代码里面输出Html，这样前端开发人员无法进行页面风格等的设计与修改，即使修改也是很麻烦，因此实际项目这种做法不可取。

#### JSP

`Java Server Page`：一种在服务器端执行的`web`组件，是一种运行在标准的`HTML`页面中嵌入脚本语言（现在只支持`Java`）的模板页面技术。本质就是在`html`代码中嵌入`java`代码。`JSP`最终还是会被编译为`Servlet`，只不过比纯`Servlet`开发页面更简单、方便。但表现逻辑、控制逻辑、业务逻辑调用还是混杂。

这种做法也是绝对不可取的，控制逻辑、表现代码、业务逻辑对象调用混杂在一起，但比直接在servlet里输出html要好一点，前端开发人员可以进行简单的页面风格等的设计与修改（但如果嵌入的java脚本太多也是很难修改的），因此实际项目这种做法不可取。

JSP本质还是Servlet，最终在运行时会生成一个Servlet（如tomcat，将在tomcat\work\Catalina\web应用名\org\apache\jsp下生成），但这种使得写html简单点，但仍是控制逻辑、表现代码、业务逻辑对象调用混杂在一起。

### Model1
可以认为是JSP的增强版，可以认为是jsp+javabean
特点：使用<jsp:useBean>标准动作，自动将请求参数封装为JavaBean组件；还必须使用java脚本执行控制逻辑。

```jsp
<%@page import="cn.javass.chapter1.javabean.UserBean"%>
<%@page language="java" contentType="text/html:charset=UTF-8"%>

<jsp:useBean id="user" class="cn.java.chaoter1.javabean.UserBean"/>
<jsp:setProperty name="user" property="*">
<%
    String submitFlag = request.getParater("submitFlag");
    if("login".equals(submitFlag)){
        if(user.login()){
        out.write("login success");
        } else {
            out.write("login fail");
        }
    } else {
%>
    <form action="" method="post">
        <input type="text" name="submitFlag" value="login"/>
        username:<input type="text" name="username"/></br>
        password:<input type="password" name="password"/></br>
        <input type="submit" value="login"/>
    </form>
```

此处我们可以看出，使用`<jsp:useBean>`标准动作可以简化`javabean`的获取/创建，及将请求参数封装到`javabean`，再看一下`Model1`架构

`Model1`架构中，`JSP`负责控制逻辑、表现逻辑、业务对象（`javabean`）的调用，只是比纯`JSP`简化了获取请求参数和封装请求参数。同样是不好的，在项目中应该严禁使用（或最多再`demo`里使用）。

## Model2

在`JavaEE`世界里，它可以认为就是`Web MVC`模型,`Model2`架构其实可以认为就是我们所说的`Web MVC`模型，只是控制器采用`Servlet`、模型采用`JavaBean`、视图采用`JSP`