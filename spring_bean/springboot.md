# springboot

### 快速入门

简单springboot项目的搭建环境步骤

* 创建maven工程
* 导入依赖坐标

```java
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.3.1.RELEASE</version>
</parent>

<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding
	<maven.compiler.source>1.8</maven.compiler.source>
	<maven.compiler.target>1.8</maven.compiler.target>
	<spring-boot.version>2.3.1.RELEASE</spring-boot.version>
</properties>

<dependencies>
	<!--web功能的起步依赖-->
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

* 编写springboot项目启动类

```Java
//该类是一个springboot引导类
@SpringBootApplication
public class MySpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(MySpringBootApplication.class);
    }
}
```

* 编写controller

```java
@Controller
public class QuickController {
    @RequestMapping("/quick")
    @ResponseBody
    public String quick() {
        System.out.println("hello 58");
        return "hello 58";
    }
}
```

* 测试
  * 浏览器输入localhost:8080/quick，打印出hello 58即为成功

### Springboot热部署

* 修改工程代码，springboot项目不需要重新启动，就可以成功
* 引入pom依赖

```java
<!--配置springboot项目热部署-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
</dependency>
```

* 修改idea配置
  * file-->settings-->complier-->build project auto...打钩
  * ctrl+shift+alt+/，regist, complier.automake.allow.when.app.running...打钩

### springboot原理分析

**依赖分析**

* springboot parent 引入了底层依赖的各种版本的jar包

* springboot web 引入了spring-web，spring-webmvc，tomcat等jar包

**自动配置**

* @SpringBootApplication注解

  * SpringBootConfiguration里面有configuration注解，表明是一个配置类
  * @ComponentScan配置包扫描
  * @EnableAutoConfiguration配置自动配置属性

  ```java
  @SpringBootConfiguration
  @EnableAutoConfiguration
  @ComponentScan(
      excludeFilters = {@Filter(
      type = FilterType.CUSTOM,
      classes = {TypeExcludeFilter.class}
  ), @Filter(
      type = FilterType.CUSTOM,
      classes = {AutoConfigurationExcludeFilter.class}
  )}
  )
  public @interface SpringBootApplication {}
  ```

* 自动配置

  ```
  @Import({AutoConfigurationImportSelector.class})
  AutoConfigurationImportSelector类里面的方法：
  	selectImports
  	getAutoConfigurationEntry
  	getCandidateConfigurations
  //读取META-INF下面的文件spring.factories
  META-INF/spring.factories
  //读取文件里面所有的autoconfiguration，例如
  ServletWebServerFactoryAutoConfiguration.calss
  //类里面引入了ServerProperties
  @EnableConfigurationProperties({ServerProperties.class})
  //ServerProperties配置了port,address等属性
  springboot里面内置的tomcat的端口属性默认在spring-configuration-metadata.json
  ```

  * springboot里面内置的tomcat的端口属性默认在spring-configuration-metadata.json

### 配置文件

首先加载yml文件，其次是yaml文件，接下来是properties

* application.yml写法

```java
# 普通数据配置
name: jtt

# 对象配置
student:
  name: jtt
  age: 26
  address: beijing

# 集合配置(String)
city:
  - beijnig
  - tianjin
  - shanghai

# 对象集合配置
studentList:
  - name: jtt
    age: 26
    address: beijing
  - name: zw
    age: 26
    address: tongzhou
```

* 读取配置文件里面的属性

  * 第一种：使用注解@Value(“${name}”)读取
  * 第二种：使用注解@ConfigurationProperties

  ```java
  <!--引入依赖-->
  <!--配置注解@configurationProperties的执行器-->
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-configuration-processor</artifactId>
  	<optional>true</optional>
  </dependency>
  
  //在代码中配置
  @ConfigurationProperties(prefix = "student")
  ```

### springboot与相关技术整合

#### 整合mybatis

* 引入依赖

```java
<!--整合mybatis-->
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>2.0.1</version>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

* 编写实体
* 编写mapper接口

```java
@Mapper
public interface AccountMapper {
    public List<Account> findAll();
}
```

* 编写mapper.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.bj58.mapper.AccountMapper">

    <select id="findAll" resultType="com.bj58.entity.Account">
        select * from account
    </select>

</mapper>
```

* 编写service和controller

* 编写配置文件application.properties

```java
//db配置
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://39.105.87.76:3306/mmall?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=admin
//mybatis配置
//起别名，加载mybatis映射文件
mybatis.type-aliases-package=com.bj58.entity
mybatis.mapper-locations=classpath:mapper/*Mapper.xml
```

#### 整合Junit

* 引入依赖

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

* 写测试类

```java
@SpringBootTest()
@RunWith(SpringRunner.class)
public class AccountControllerTest {}
//此处需注意引入test包的正确性import org.junit.Test;
```

#### 整合spring data jpa

* 引入依赖

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
	<version>2.0.0.RELEASE</version>
</dependency>
//同时需要mysql依赖
```

* 编写配置文件application.properties

```java
//数据库配置
#jpa配置
spring.jpa.database=mysql
spring.jpa.show-sql=true
spring.jpa.generate-ddl=true
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming.implicit-strategy=org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
```

* 编写实体类

```java
@Entity
public class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

* 编写repository接口

```java
public interface AccountRepository extends JpaRepository<Account, Integer> {
    public List<Account> findAll();
}
```

* 编写service和controller

#### 整合Redis

* 引入依赖

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

* 编写配置文件application.properties

```java
#redis配置
spring.redis.host=39.105.87.76
spring.redis.port=6379
# 连接超时时间（毫秒）
spring.redis.timeout=5000
```

* 编写业务类service

```java
@Service
public class AccountService {
    @Autowired
    private AccountMapper accountMapper;
    @Autowired
    private RedisTemplate redisTemplate;

    public String findAll() throws JsonProcessingException {
        //从redis缓存中获取
        String accountListJson = redisTemplate.boundValueOps("account.findAll").get().toString();
        //获取不到，从数据库查询
        if (accountListJson == null) {
            List<Account> all = accountMapper.findAll();
            ObjectMapper objectMapper = new ObjectMapper();
            accountListJson = objectMapper.writeValueAsString(all);
            //查询到之后，写入到redis缓存中
            redisTemplate.boundValueOps("account.findAll").set(accountListJson);
            System.out.println("从数据库查询...");
        } else {
            System.out.println("从redis缓存中获取...");
        }
        return accountListJson;
    }
}
```

经测试，第一次从数据库查询，第二次从redis缓存中查找。