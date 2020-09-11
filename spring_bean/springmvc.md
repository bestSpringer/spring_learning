# SpringMVC进阶

* 组件介绍
  * 前端控制器 DispatcherServlet
  * 处理器映射器 HandlerMapping
  * 处理器适配器 HandlerAdapter
  * 处理器 Handler
  * 视图解析器 View Resolver
* 参数绑定
	* 请求参数绑定
    * @requestMapping URL路径与方法对应关系（方法和类上）
      * 属性：value,path,method,params 

```java
<web.xml>
<!--配置前端控制器-->
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

<!--配置中文乱码过滤器-->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>["/*"]</url-pattern>
</filter-mapping>

<!--配置spring监听器,默认只加载WEB-INF目录下的applicationContext.xml配置文件-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

```java
<springmvc.xml>
<!--包扫描-->
<context:component-scan base-package="com.itheima.bj"/>

<!--视图解析器-->
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/pages/"/>
    <property name="suffix" value=".jsp"/>
</bean>

<!--类型转换器-->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.itheima.bj.utils.StringToDateConverter"></bean>
        </set>
    </property>
</bean>

<!--开启mvc注解，包含了处理器映射器，处理器适配器-->
<mvc:annotation-driven conversion-service="conversionService"/>

<!--前端控制器，配置哪些资源不拦截-->
<mvc:resources mapping="/css/" location="/css/**"/>
<mvc:resources mapping="/images/" location="/images/**"/>
<mvc:resources mapping="/js/" location="/js/**"/>
```

### 参数绑定

请求参数绑定

- @RequestMapping 

  * URL路径与方法对应关系（方法和类上）

  - 属性：value,path,method,params 

* @RequestParam

```java
/**
* 参数为基本类型
* 属性：value,name
* required为 true，必填
* defaultValue：默认值
*/
public String hello(@RequestParam Long id){}
```

* @RequestBody
  * 传参为java对象dto对象

```java
/**
* 参数为java对象
* 属性：required为 true，必填
*/
public String hello(@RequestBody() HelloReq req) {}
```

* @PathVariable
  * 路径参数url

```java
/**
* 属性：value,name
* required为 true，必填
*/
@RequestMapping("/hello/{id}")
public String hello(@PathVariable Long id) {}
```

* @RequestHeader

```java
/**
* @RequestHeader获取请求头信息
*/
@RequestMapping("/hello")
public String hello(@RequestHeader(value = "Accept") String header) {
    System.out.println(header);
    return "success";
}
```

* @CookieValue

```java
/**
* @CookieValue获取cookie的值
*/
@RequestMapping("/hello")
public String hello1(@CookieValue(value = "JSESSIONID") String cookieValue) {
     System.out.println(cookieValue);
     return "success";
}
```

* 返回值为void

```java
@RequestMapping("/hello")
public void hello(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	System.out.println("hello jingdong");
	//请求转发到success
	request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request, response);
	//请求重定向到success
	response.sendRedirect(request.getContextPath() + "/index.jsp");
	//请求直接进行响应,解决中文乱码
	response.setCharacterEncoding("UTF-8");
	response.setContentType("text/html;charset=UTF-8");
	response.getWriter().print("张伟");
	return;
}
```

```java
//使用关键字的方式进行转发或者重定向
public String forwardOrRedirect() {
	//请求转发
	return "forward:/WEB-INF/pages/success.jsp";
	//请求重定向
	return "redirect:/index.jsp";
}
```

响应

* @ResponseBody

```java
<!--前端控制器，配置哪些资源不拦截-->
<mvc:resources mapping="/css/" location="/css/**"/>
<mvc:resources mapping="/images/" location="/images/**"/>
<mvc:resources mapping="/js/" location="/js/**"/>
```

### springmvc实现文件上传

* 传统方式上传文件

  * 引入依赖

    ```java
    <dependency>
       <groupId>commons-fileupload</groupId>
       <artifactId>commons-fileupload</artifactId>
       <version>1.2.2</version>
    </dependency>
    <dependency>
       <groupId>commons-io</groupId>
       <artifactId>commons-io</artifactId>
       <version>2.6</version>
    </dependency>
    ```

  * 上传文件

    ```java
    @RequestMapping("/fileupload")
    public String fileupload(HttpServletRequest request) throws Exception {
        System.out.println("文件上传...");
        String realPath = request.getSession().getServletContext().getRealPath("/uploads/");
        System.out.println(realPath);
        File file = new File(realPath);
        if (!file.exists()) {
            file.mkdirs();
        }
        DiskFileItemFactory dfi = new DiskFileItemFactory();
        ServletFileUpload servletFileUpload = new ServletFileUpload(dfi);
        //解析request，获得文件项
        List<FileItem> items = servletFileUpload.parseRequest(request);
        for (FileItem item : items) {
            //如果此文件项为上传文件项
            if (!item.isFormField()) {
                //获取文件名
                String name = item.getName();
                item.write(new File(realPath, name));
                //上传文件大于10KB，会产生临时文件，这里需要删除临时文件
                item.delete();
            }
        }
        return "success";
    }
    ```

* springmvc上传文件

  * springmvc上传文件原理

  ![springmvc_fileupload](C:\Users\jiatiantian\Desktop\springmvc插图\springmvc_fileupload.png)

  * 配置文件解析器对象

  ```java
  <springmvc.xml>
  <!--配置文件解析器对象-->
      <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
          <property name="maxUploadSize" value="10485760"/>
      </bean>
  ```

  * 上传文件

  ```java
  @RequestMapping("/fileupload2")
      public String fileupload2(HttpServletRequest request, MultipartFile upload) throws Exception {
          System.out.println("springmvc文件上传...");
          String filepath = request.getSession().getServletContext().getRealPath("/uploads/");
          System.out.println(filepath);
          File file = new File(filepath);
          if (!file.exists()) {
              file.mkdirs();
          }
          String filename = upload.getOriginalFilename();
          String uuid = UUID.randomUUID().toString().replace("-", "");
          filename = uuid + "_" + filename;
          upload.transferTo(new File(filepath, filename));
          return "success";
      }
  ```

  ### springmvc异常处理

  编写异常处理器

  ```java
  //实现HandlerExceptionResolver接口
  public class SysExceptionResolver implements HandlerExceptionResolver{
      //实现方法resolveException
  }
  //在springmvc.xml注册bean对象
  ```

  ### springmvc拦截器

  自定义拦截器

  ```java
  //实现HandlerInterceptor
  public class MyInterceptor implements HandlerInterceptor{
      //重写preHandle，预处理，在controller执行之前处理
      //return true放行，执行下一个拦截器，如果没有下一个拦截器，执行controller中的方法
      //return flase拦截，处理之后放行
      public boolean preHandle(request,response,handler) throws Exception{
          System.out.println("ssdda");
          return true;
      }
      //postHandle方法，执行controller方法-->执行postHandle方法-->执行jsp方法-->执行afterCompletion方法
  }
  ```

  配置拦截器

  ```java
  <!--配置拦截器-->
      <mvc:interceptors>
          <mvc:interceptor>
              <!--要拦截的具体方法,"/**"拦截所有方法-->
              <mvc:mapping path="/hello/*"/>
              <!--不拦截的方法-->
              <!-- <mvc:exclude-mapping path="/user/*"/>-->
              <!--配置拦截器对象-->
              <bean class="com.itheima.bj.interceptor.MyInterceptor"/>
          </mvc:interceptor>
      </mvc:interceptors>
  ```

  拦截器的执行顺序：

  * 执行preHandle方法
  * 执行controller方法
  * 执行postHandle方法
  * 执行jsp
  * 执行afterCompletion方法

## spring websocket消息中间件

**HTTP长连接与短连接**
* 短连接的操作步骤是：
建立连接——数据传输——关闭连接...建立连接——数据传输——关闭连接
* 长连接的操作步骤是：
建立连接——数据传输...（保持连接）...数据传输——关闭连接
HTTP长连接发送http请求后服务端如果没有返回就一直连着，等服务端有东西了推送给浏览器，相当于给之前的请求一个响应，http连接断了。然后浏览器在发送一个请求，服务器响应，断开，循环往复。
HTTP协议规定了http连接是一个一来一回的过程。一个请求获得一个响应后必须断掉，而且只有先有请求后有响应。
websocket建立连接需要先通过一个http请求进行和服务器端握手，握手通过后连接就建立并保持了。浏览器和服务端可以用这个连接相互发送请求了。

**spring websocket 利用注解接收和发送消息**

STOMP(Streaming Text Orientated Message Protocol)流文本定向消息协议

* 是一种为MOM(Message Oriented Middleware，面向消息的中间件)设计的简单文本协议。

* 消息格式为：

```
COMMAND
header1:value1
header2:value2
Body^@
```

  * eg:

    * 发送消息
    
      ```
      SEND
			destination:/queue/trade
			content_type:application/json
			content_length:44
			{"action":"BUY","ticker":"MMM","shares",44}^@
		```
    
    * 订阅消息
    
    ```
    SUBSCRIBE
    id:sub-1
        destination:/topic/price.stock.*
        ^@
    ```
    
    * 服务器广播消息
    

```
MESSAGE
      message-id:nxahklf6-1
      subscription:sub-1
      destination:/topic/price.stock.MMM
	    {"ticker":"MMM","price":129.45}^@
```

spring websocket利用STOMP作为websocket的子协议，原因是stomp可以提供一种类似springmvc的编码方式，可以悠闲地俄利用注解进行接收消息和发送消息。

```java
@Controller
@RequestMapping("/websocket")
@MessageMapping("foo")
public class WebScoketController {
	/**
	 * @MessageMapping用在方法上或者类上，
	 * 与springmvc不同的是，springmvc的路径是类路径/方法，websocket使用.来分开路径的，如foo.handler;
	 * @SendTo和@SendToUser进行广播推送和精准推送
	 * @SendTo可以把消息广播到路径上去;
	 */
    @MessageMapping("handler")
    @SendTo("/topic/greetings")
    public String handler(Task task, String name) {
        //...
        return "success";
    }
}
```

```java
@Configuration
@EnableWebSocketMessageBroker
public class ConfigWebSocket implements WebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/websocket/topic"); // 客户端订阅地址前缀
        config.setApplicationDestinationPrefixes("/websocket"); // 客户端发送消息地址前缀
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/websocket/connect")
                .addInterceptors(new HttpSessionHandshakeInterceptor()) // 会把 HttpSession 中的属性拷贝到 MessageHeaders 里
                .setAllowedOrigins("*") // 允许跨域
                .withSockJS();
    }
}
```

* Message message：最基础的消息体，含有header和payload信息
* @Payload 消息体内容
* @Header(“..”) 某个头部key的值
* @Headers, 所有头部key的map集合	
* MessageHeaders , SimpMessageHeaderAccessor, MessageHeaderAccessor ,StompHeaderAccessor 消息头信息
* @DestinationVariable 类似springmvc中的@PathVariable
* @MessageExceptionHandler消息中间件的异常处理