---
title: Jetty server Kerberos Authentication
date: 2017-08-31 16:10:59
tags: [Java, Jetty, SPNEGO, Linux, Kerberos, HttpClient]
---

## 预备步骤
1. 在Windows Active Diectory 中注册SPN， 创建Keytab file
2. 准备以下3个配置文件：
    1. krb5.ini
    ```
    [libdefaults]
        default_realm    = EXAMPLE.NET
        forwardable      = true
        dns_lookup_realm = false
        dns_lookup_kdc   = false
        ticket_lifetime  = 24h

    [realms]
        EXAMPLE.NET = {
               kdc = dc1.example.net:88
               kdc = dc2.example.net:88
        }

    [domain_realm]
        .exmple.net = EXAMPLE.NET

    [logging]
        default      = FILE:./logs/krb5libs.log
        kdc          = FILE:./logs/kdc.log
        admin_server = FILE:./logs/kadmind.log
    ```

    2. spnego.properties
    ```
    targetName = HTTP/server1.example.net
    ```
    3. spnego.conf
    ```
    com.sun.security.jgss.initiate {
        com.sun.security.auth.module.Krb5LoginModule required
        doNotPrompt=true
        principal="HTTP/server1.example.net@EXAMPLE.NET"
        useKeyTab=true
        keyTab="./conf/krb5.keytab"
        storeKey=true
        isInitiator=false
        debug=true;
    };

    com.sun.security.jgss.accept {
        com.sun.security.auth.module.Krb5LoginModule required
        doNotPrompt=true
        principal="HTTP/server1.example.net@EXAMPLE.NET"
        useKeyTab=true
        keyTab="./conf/krb5.keytab"
        storeKey=true
        isInitiator=false
        debug=true;
    };
    ```

## 在Jetty Server中添加Kerberos Authentication
### 使用Jetty SpnegoLoginService, 基于配置文件的方法
1. 在命令行定义以下环境变量, 指定krb5和spnego配置文件的位置：
``` java
-Djava.security.krb5.conf=/path/to/jetty/etc/krb5.ini \
-Djava.security.auth.login.config=/path/to/jetty/etc/spnego.conf \
-Djavax.security.auth.useSubjectCredsOnly=false
```
2. 如果需要debug的话，也可以定义下面的变量：
``` java
-Dorg.eclipse.jetty.LEVEL=debug \
-Dsun.security.spnego.debug=all
```
3. 在web.xml中添加以下内容
```xml
 <security-constraint>
   <web-resource-collection>
     <web-resource-name>Secure Area</web-resource-name>
     <url-pattern>/*</url-pattern>
   </web-resource-collection>
   <auth-constraint>
     <!-- this is the domain that the user is a member of -->
     <role-name>EXAMPLE.NET</role-name>
   </auth-constraint>
 </security-constraint>
 <login-config>
   <auth-method>SPNEGO</auth-method>
   <realm-name>Test Realm</realm-name>
 </login-config>
```
4. 需要在jetty.xml 或者webapp的context file中创建一个UserRealm
Jetty xml文件中的配置如下：
```
  <Call name="addBean">
     <Arg>
       <New class="org.eclipse.jetty.security.SpnegoLoginService">
         <Set name="name">Test Realm</Set>
         <Set name="config"><Property name="jetty.home" default="."/>/etc/spnego.properties</Set>
       </New>
     </Arg>
   </Call>
```
context file中的配置如下：
 ```
 <Get name="securityHandler">
   <Set name="loginService">
     <New class="org.eclipse.jetty.security.SpnegoLoginService">
       <Set name="name">Test Realm</Set>
       <Set name="config">
        <SystemProperty name="jetty.home" default="."/>/etc/spnego.properties
      </Set>
     </New>
   </Set>
   <Set name="checkWelcomeFiles">true</Set>
 </Get>
 ```
### 使用Jetty SpnegoLoginService, 基于代码的方法
使用单独的文件web_auth.xml处理Kerberos相关的配置，可以把Kerberos Authentication变成configurarble
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_ID" version="3.0"
         xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_3.0.xsd">
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Secure Area</web-resource-name>
            <url-pattern>/*</url-pattern>
            <http-method>POST</http-method>
            <http-method>GET</http-method>
        </web-resource-collection>
        <auth-constraint>
            <!-- this is the domain that the user is a member of -->
            <role-name>EXAMPLE.NET</role-name>
        </auth-constraint>
    </security-constraint>

    <security-role>
        <role-name>EXAMPLE.NET</role-name>
    </security-role>

    <login-config>
        <auth-method>SPNEGO</auth-method>
        <realm-name>Kerberos</realm-name>
    </login-config>
</web-app>
```
```java
public static void main(String[] args) throws Exception {

        //For Https, replace with SslSelectChannelConnector
        SelectChannelConnector connector = new SelectChannelConnector();
        connector.setPort(port);
        connector.setAcceptors((Runtime.getRuntime().availableProcessors())/2);

        Server server = new Server();
        server.addConnector(connector);

        WebAppContext context = new WebAppContext();
        context.setDescriptor("./web.xml");
        context.setResourceBase(".");
        context.setContextPath("/");
        context.setParentLoaderPriority(true);

        boolean enabledAuth = <enabledAuth>;
        log.info("Enabled Kerberos Authentication? " + enabledAuth);
        if (enabledAuth) {
            System.setProperty("java.security.krb5.conf", <path to krb5.ini>);
            System.setProperty("java.security.auth.login.config", <path to spnego.conf>);
            System.setProperty("javax.security.auth.useSubjectCredsOnly", "false");

            context.addOverrideDescriptor("./web_auth.xml");
            SecurityHandler sh = context.getSecurityHandler();

            SpnegoLoginService loginService = new SpnegoLoginService();
            loginService.setName("Kerberos");
            loginService.setConfig(<path to spnego.properties>);

            sh.setLoginService(loginService);
            server.addBean(loginService);
        }

        server.setHandler(context);

        server.start();
        server.join();
    }
```

### 使用source forge SpnegoHttpFilter的方法
背景前提是测试环境的Windows server不在同一个域中，无法从AD获取到token。当时有人问了一下能不能创建一个IP地址的白名单，对这些IP不做验证。

Jetty SpnegoLoginService做验证的时机是在调用ServletFilterChain之前验证，如果验证失败就返回401Error, 不会继续调用Filter了。 使用SpnegoHttpFilter的话就会比较灵活，可以在进行认证前做一些操作，比如判断是可信的IP地址的话就bypass authentication.

SpnegoHttpFilter的Maven dependency：
```xml
<dependency>
    <groupId>net.sourceforge.spnego</groupId>
    <artifactId>spnego</artifactId>
    <version>7.0</version>
</dependency>
```

同样所需的配置文件如下：
1. login.conf
```
spnego-client {
    com.sun.security.auth.module.Krb5LoginModule required;
};

spnego-server {
    com.sun.security.auth.module.Krb5LoginModule required
    doNotPrompt=true
    principal="HTTP/server1.example.net@EXAMPLE.NET"
    useKeyTab=true
    keyTab="<path to keytab>"
    storeKey=true
    isInitiator=false
    debug=true;
};
```
2. 在web.xml中添加filter和filter mapping
```xml
    <filter>
        <filter-name>SpnegoHttpFilter</filter-name>
        <filter-class>net.sourceforge.spnego.SpnegoHttpFilter</filter-class>

        <init-param>
            <param-name>spnego.allow.basic</param-name>
            <param-value>true</param-value>
        </init-param>

        <init-param>
            <param-name>spnego.allow.unsecure.basic</param-name>
            <param-value>true</param-value>
        </init-param>

        <init-param>
            <param-name>spnego.prompt.ntlm</param-name>
            <param-value>true</param-value>
        </init-param>

        <init-param>
            <param-name>spnego.login.client.module</param-name>
            <param-value>spnego-client</param-value>
        </init-param>

        <init-param>
            <param-name>spnego.krb5.conf</param-name>
            <param-value>krb5.conf</param-value>
        </init-param>

        <init-param>
            <param-name>spnego.login.conf</param-name>
            <param-value>login.conf</param-value>
        </init-param>

        <init-param>
            <param-name>spnego.login.server.module</param-name>
            <param-value>spnego-server</param-value>
        </init-param>

        <init-param>
            <param-name>spnego.logger.level</param-name>
            <param-value>1</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>SpnegoHttpFilter</filter-name>
        <url-pattern>/Hello</url-pattern>
    </filter-mapping>
```

### 创建测试页面
这里我们创建一个简单的测试用的Servlet， 当Kerberos 验证通过后，Server是可以拿到User Principal的：
```java
public class HelloServlet extends javax.servlet.http.HttpServlet {

    private static final long serialVersionUID = -2626689137720986094L;

    public HelloServlet() {
        super();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String responseXML = String.format("<h1>Kerberos Tester Page, Hello %s</h1>", getUserPrincipal(request));
        byte[] bytes = responseXML.getBytes();
        response.setBufferSize(bytes.length);
        ServletOutputStream out = response.getOutputStream();
        out.write(bytes);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }

    private String getUserPrincipal(HttpServletRequest servletRequest) {
        //For Jetty SpnegoLoginService
        if (servletRequest instanceof Request) {
            Request request = (Request) servletRequest;
            Authentication authentication = request.getAuthentication();
            if (authentication instanceof UserAuthentication) {
                UserAuthentication userAuth = (UserAuthentication) authentication;
                return userAuth.getUserIdentity().getUserPrincipal().getName();
            }
        }
        //For SpnegoHttpFilter
        return servletRequest.getUserPrincipal() == null ? "Unknown User" : servletRequest.getUserPrincipal().getName();
    }
}
```
![](http://res.cloudinary.com/delong/image/upload/v1505986404/kerberos_tester_zctiqt.png)

### Java Client访问带有Keberos Authentication的web service
使用Apache的WinHttpClient来进行Kerberos Authentication, 底层会通过jna去掉用windows的api去KDC获取当前已经登录的用户的token
Maven dependency:
```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient-win</artifactId>
    <version>4.5.3</version>
</dependency>
```
```java
import org.apache.http.HttpEntity;
import org.apache.http.HttpHost;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.ByteArrayEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.WinHttpClients;
import org.apache.http.util.EntityUtils;


public class HttpClientAuthentication {

    public final static void main(String[] args) throws Exception {

        if (!WinHttpClients.isWinAuthAvailable()) {
            System.out.println("Integrated Win auth is not supported!!!");
        }

        CloseableHttpClient httpclient = WinHttpClients.createDefault();
        String url = "<url>";
        // There is no need to provide user credentials
        // HttpClient will attempt to access current user security context through
        // Windows platform specific methods via JNA.
        try {
            HttpGet request = new HttpGet(url);

            System.out.println("Executing request " + request.getRequestLine());
            CloseableHttpResponse response = httpclient.execute(request);
            try {
                System.out.println("----------------------------------------");
                System.out.println(response.getStatusLine());
                System.out.println(EntityUtils.toString(response.getEntity()));
                EntityUtils.consume(response.getEntity());
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }
    }
}
```
![](http://res.cloudinary.com/delong/image/upload/v1506050792/kerberos_http_client_ymdboq.png)
## Links
[Introduction to JAAS and Java GSS-API Tutorials](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jgss/tutorials/index.html)

[常见的Kerboros错误信息](http://blog.csdn.net/zy531/article/details/43327863)

[Use of JAAS Login Utility and Java GSS-API for Secure Message Exchanges](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jgss/tutorials/ClientServer.html)

[Use of Java GSS-API for Secure Message Exchanges *Without* JAAS Programming](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jgss/tutorials/BasicClientServer.html)

[Jetty Authentication](https://www.eclipse.org/jetty/documentation/9.4.x/configuring-security-authentication.html)

[Jetty Spnego Support](https://www.eclipse.org/jetty/documentation/9.4.x/spnego-support.html)

![](http://res.cloudinary.com/delong/image/upload/v1503370246/KerberosExplained_kwxc4k.png)