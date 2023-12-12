# SpringSecurity

> 在 Spring 3.1 中，对Java Configuration的常规支持已添加到 Spring Framework 中。从 Spring Security 3.2 开始，Spring Security Java Configuration 支持使用户可以轻松配置 Spring Security，而无需使用任何 XML。

### 1.Hello Web Security Java 配置

第一步是创建我们的 *Spring Security Java* 配置。该配置将创建一个称为springSecurityFilterChain的 Servlet 过滤器，该过滤器负责应用程序中的所有安全性(保护应用程序 URL，验证提交的用户名和密码，重定向到登录表单等)。您可以在下面找到 Spring Security Java 配置的最基本示例：
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.*;
import org.springframework.security.config.annotation.authentication.builders.*;
import org.springframework.security.config.annotation.web.configuration.*;

@EnableWebSecurity
public class WebSecurityConfig implements WebMvcConfigurer {

    @Bean
    public UserDetailsService userDetailsService() throws Exception {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withDefaultPasswordEncoder().username("user").password("password").roles("USER").build());
        return manager;
    }
}
```
此配置的确没有太多，但是它做了很多。您可以找到以下功能的摘要：

* 要求对应用程序中的每个 URL 进行身份验证
* 为您生成一个登录表单
* 允许具有 **Username** 和 **Password** 的用户使用基于表单的身份验证进行身份验证
* 允许用户注销
* CSRF 攻击 prevention
* Session Fixation 保护
* 安全标题集成
* HTTP 严格传输安全安全请求
    * X-Content-Type-Options integration
    * 缓存控制(以后可以由您的应用程序覆盖以允许缓存您的静态资源)
    * X-XSS-Protection integration
    * X-Frame-Options 集成有助于防止Clickjacking
    * 与以下 Servlet API 方法集成
    * HttpServletRequest#getRemoteUser()
    * HttpServletRequest.html#getUserPrincipal()
    * HttpServletRequest.html#isUserInRole(java.lang.String)
    * HttpServletRequest.html#login(java.lang.String, java.lang.String)
    * HttpServletRequest.html#logout()

#### 1.1. AbstractSecurityWebApplicationInitializer

下一步是向 War 注册springSecurityFilterChain。可以在 Servlet 3.0 环境中使用Spring 的 WebApplicationInitializer 支持在 Java 配置中完成此操作。毫不奇怪，Spring Security 提供了一个 Base ClassAbstractSecurityWebApplicationInitializer，它将确保springSecurityFilterChain为您注册。使用AbstractSecurityWebApplicationInitializer的方式因我们是否已经在使用 Spring 或 Spring Security 是应用程序中唯一的 Spring 组件而异。

* 第 6.1.2 节“不存在 Spring 的 AbstractSecurityWebApplicationInitializer”-如果您尚未使用 Spring，请按照以下说明进行操作
* 第 6.1.3 节“带有 Spring MVC 的 AbstractSecurityWebApplicationInitializer”-如果您已经在使用 Spring，请按照以下说明进行操作


##### 1.2.1 不存在 Spring 的 AbstractSecurityWebApplicationInitializer

如果不使用 Spring 或 Spring MVC，则需要将WebSecurityConfig传递到超类中，以确保配置被接受。您可以在下面找到一个示例：
```java
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer
    extends AbstractSecurityWebApplicationInitializer {

    public SecurityWebApplicationInitializer() {
        super(WebSecurityConfig.class);
    }
}
```
SecurityWebApplicationInitializer将执行以下操作：
为应用程序中的每个 URL 自动注册 springSecurityFilterChain 过滤器
添加一个加载WebSecurityConfig的 ContextLoaderListener。

##### 1.2.2 使用 Spring MVC 的 AbstractSecurityWebApplicationInitializer

如果我们在应用程序的其他地方使用 Spring，则可能已经有一个WebApplicationInitializer用来加载 Spring 配置。如果我们使用以前的配置，将会得到一个错误。相反，我们应该使用现有的ApplicationContext注册 Spring Security。例如，如果我们使用 Spring MVC，则SecurityWebApplicationInitializer如下所示：
```java
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer
    extends AbstractSecurityWebApplicationInitializer {

}
```

这只会为应用程序中的每个 URL 仅注册 springSecurityFilterChain 过滤器。之后，我们将确保WebSecurityConfig已加载到我们现有的 ApplicationInitializer 中。例如，如果我们使用的是 Spring MVC，则将其添加到getRootConfigClasses()
```java
public class MvcWebApplicationInitializer extends
        AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[] { WebSecurityConfig.class };
    }

    // ... other overrides ...
}
```
### 2. HttpSecurity

到目前为止，我们的WebSecurityConfig仅包含有关如何验证用户身份的信息。 Spring Security 如何知道我们要要求所有用户进行身份验证？ Spring Security 如何知道我们要支持基于表单的身份验证？原因是WebSecurityConfigurerAdapter在configure(HttpSecurity http)方法中提供了默认配置，如下所示
```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .anyRequest().authenticated()
        .and()
        .formLogin()
        .and()
        .httpBasic();
}
```

上面的默认配置：
确保对我们应用程序的任何请求都需要对用户进行身份验证

* 允许用户使用基于表单的登录进行身份验证
* 允许用户使用 HTTP Basic 身份验证进行身份验证

您会注意到该配置与 XML 命名空间配置非常相似：
```xml
<http>
    <intercept-url pattern="/**" access="authenticated"/>
    <form-login />
    <http-basic />
</http>
```

使用and()方法表示等效于关闭 XML 标记的 Java 配置，该方法允许我们 continue 配置父级。如果您阅读该代码，这也很有意义。我要配置授权请求，并配置表单登录，并配置 HTTP 基本身份验证。

### 3. Java 配置和表单登录

您可能想知道提示您登录时登录表单的来源，因为我们没有提及任何 HTML 文件或 JSP。由于 Spring Security 的==默认配置没有显式设置登录页面的 URL==，因此 Spring Security 将基于已启用的功能并使用处理提交的登录的 URL 的标准值(用户将使用的默认目标 URL)自动生成一个 URL。登录后发送至。\
尽管自动生成的登录页面便于快速启动和运行，但是大多数应用程序都希望提供自己的登录页面。为此，我们可以如下所示更新配置：
```java
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        .anyRequest().authenticated()
        .and()
        .formLogin()
        .loginPage("/login") (1)
        .permitAll();        (2)
}
```
* (1) 更新后的配置指定登录页面的位置。
* (2) 我们必须授予所有用户(即未经身份验证的用户)访问我们登录页面的权限。 formLogin().permitAll()方法允许向所有用户授予与基于表单的登录相关的所有 URL 的访问权限。

下面是我们当前配置中使用 JSP 实现的示例登录页面：
```html
<c:url value="/login" var="loginUrl"/>
<form action="${loginUrl}" method="post">       (1)
    <c:if test="${param.error != null}">        (2)
        <p>
            Invalid username and password.
        </p>
    </c:if>
    <c:if test="${param.logout != null}">       (3)
        <p>
            You have been logged out.
        </p>
    </c:if>
    <p>
        <label for="username">Username</label>
        <input type="text" id="username" name="username"/>  (4)
    </p>
    <p>
        <label for="password">Password</label>
        <input type="password" id="password" name="password"/>  (5)
    </p>
    <input type="hidden"                        (6)
        name="${_csrf.parameterName}"
        value="${_csrf.token}"/>
    <button type="submit" class="btn">Log in</button>
</form>
```

* (1) 到/login URL的 POST 将尝试验证用户身份
* (2) 如果查询参数error存在，则尝试认证失败
* (3) 如果查询参数logout存在，则表明用户已成功注销
* (4) 用户名必须作为名为 username 的 HTTP 参数存在
* (5) 密码必须作为名为 password 的 HTTP 参数存在
* (6) 我们必须名为“包含CSRF令牌”的部分要了解更多信息，请阅读参考的第 10.6 节“跨站请求伪造(CSRF)”部分

### 4. 授权请求
