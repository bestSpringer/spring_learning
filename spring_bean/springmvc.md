# SpringMVC进阶









## spring websocket消息中间件

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
