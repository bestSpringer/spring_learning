# Spring进阶

## bean

**ac 常用实现类**

ApplicationContext的三个常用实现类：

* ClassPathXmlApplicationContext:加载类路径下的配置文件
* FileSystemXmlApplicationContext:加载任意磁盘下的配置文件 
* AnnotationConfigApplicationContext:用于读取注解创建容器的

**Spring 核心容器的两个接口引发的问题**

1. ApplicationContext: （单例模式适用）

   ​	它在构建核心容器时，创建对象的策略是采用立即加载的方式。也就是说，只要一读取完配置文件马上传建配置文件中的配置的对象。在执行完 new ClassPathXmlApplicationContext("配置文件")后，利用反射创建对象。

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
IUserService userService = ac.getBean("userService", UserServiceImpl.class);
userService.save();
```

2. BeanFactory：（多例模式适用）

   ​	它在构建核心容器时，创建对象采用的策略是采用延迟加载的方式。也就是说，什么时候根据key获取对象了，什么时候才真正创建对象。

```java
Resource resource = new ClassPathResource("bean.xml");
BeanFactory bean = new XmlBeanFactory(resource);
UserServiceImpl userService = bean.getBean("userService", UserServiceImpl.class);
userService.save();
```

**Bean 的作用范围**

scope属性：

* singleton：单例的（默认）
* prototype：多例的
* request：web应用的请求范围
* session：web应用的会话范围
* global-session：集群环境的会话范围（全局会话范围）

**Bean 的生命周期**

* 单例：容器在，对象在
* 多例：
  * 出生：适用对象时，创建对象
  * 或者：对象只要在使用
  * 死亡：对象长时间不用，且没有别的对象引用时，由java的垃圾回收机制回收

**Bean 依赖注入**

* 可以注入的数据：基本类型和String、其他bean类型，在配置文件中配置的、复杂类型
* 注入的方式：
  * 使用构造函数
  * 使用set注入
  * 使用注解注入

***方式一：使用构造函数注入***

* 使用的标签：constructor-arg    bean标签的内部

* 标签中的属性：
  * type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或者某些参数的类型
  * index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置是从0开始的
  * name：用于指定给构造函数中指定名称的参数赋值
  * value：用于提供基本类型和String类型的数据
  * ref：用于指定其他的bean类型数据，指的是在Spring的IOC核心容器中出现过的bean对象

* 优点：在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。
* 缺点：改变了bean对象的实例化方式，在创建对象时，如果用不到这些数据，也必须提供。

***方式二：set方法注入***

* 使用的标签：property   （bean标签的内部）
* 标签中的属性：
  * name：用于指定注入时所调用的set方法名称
  * value：用于提供基本类型和String类型的数据
  * ref：用于指定其他的bean类型数据，指的是在Spring的IOC核心容器中出现过的bean对象
* 优点：创建对象时，没有明确的限制，可以直接使用默认构造函数
* 缺点：如果有某个成员有值，则获取对象是有可能set方法没有执行

***方式三：复杂类型/集合类型注入***

```java
<property name="myString">
	<array>AAA</array>
</property>
```

* 用于给list结构集合注入的标签：list,array,set
* 用于给map结构集合注入的标签：map,props
* 结构相同，可以互换

### 四类注解

​		**用于创建对象的注解**

* @Component ：用于把当前类对象存入Spring容器中 
  * value属性：用于指定bean的id
    * 注意：（不属于三层的其他需要注入到容器中的对象，要用@Component）
    * @Controller：表现层
    * @Service：业务层
    * @Repository：持久层

​		**用于注入数据的注解**

* @Autowired：自动按照类型注入
  * @Qualifier
    * 在按照类型注入的基础上，在按照名称注入
    * 在给类成员注入时，不能单独使用，在给方法参数注入时可以
    * value属性：用于指定bean的id
  * @Resource
    * 直接按照bean的id注入，可以独立使用
    * name属性，用于指定bean的id
    * 集合类型的注入只能通过xml来实现
    * 基本类型和String类型的注入：
      * @Value：value，用于指定数据的值。可以使用SpEL表达式（${表达式}）

​		**用于改变作用范围的注解**

* @Scope：用于指定bean的作用范围
  * value：取值singleton和prototype

​		**用于生命周期的注解**

###### **用配置类干掉xml**

* @Configuration：指定此类是一个配置类
* @ComponentScan：用于通过注解指定spring在创建容器时要扫描的包

```
@Configuration
@ComponentScan(basePackages = "com.itheima")
```

* @Bean：用于把当前方法的返回值作为bean对象存入spring容器中
  * name属性，用于指定bean的id
  * 细节：使用注解配置方式时，如果方法有参数，spring会去容器中查找有没有可用的bean对象，查找的方式和autowired方法一样。

* @Import(JdbcConfig.class)：用于导入其他的配置类
  * value属性：用于指定其他配置类的字节码，当使用Import注解后，有Import注解配置类是主配置类，其他的导入的配置类为子配置类。
  * 配置类不需要写@Configuration,也不需要写扫描的包

```java
//注解
	Application ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
```

* 抽取出jdbc配置

  * 文件名：jdbc.properties
```java
//用于写在jdbc配置类上，指定properties文件的位置
//value属性：指定文件的名称和路径
//classpath：表示类路径下
@PropertySource("classpath:jdbc.properties")
@Value("${jdbc.driver}")
private String driver;
```

* 父子配置类的配置

```java
/**
 * 父子配置类
 * 要么用 @Import 导入子配置类
 * 要么在子配置类上 @Configuration，并且在主配置类上配置包扫描
 * 子配置类要引入properties文件的东西，需要加@PropertySource("classpath:jdbc.properties")
 */
//表明是一个配置类
@Configuration
//包扫描
@ComponentScan(basePackages = "com.itheima")
//导入子配置类
@Import({JdbcConfig.class, TransactionConfig.class})
//开始事务
@EnableTransactionManagement
//开始AOP
//@EnableAspectJAutoProxy

public class SpringConfiguration {}
```

```java
/**
 * 属性：classes 基于注解类的
 * location基于xml文件的
 * 测试类配置：引入spring-test坐标以及junit坐标
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
```

### AOP:使用动态代理技术

* 术语：
  * Joinpoint连接点（业务层方法）
  * Pointcut切入点
  * 注意：所有的切入点都是连接点，但是并不是所有的切入点都是连接点。业务层的方法被增强是连接点。

**Spring中基于XML的AOP配置步骤**

1. 把通知bean交给spring管理
2. 使用aop:config标签表名开始AOP的配置
3. 使用aop:aspect标签表名配置切面
   * 属性：
     * id是给切面提供的一个唯一标识
     * ref是指定通知类bean的id
4. 在aop:aspect标签的内部使用对应标签来配置通知的类型
   * 示例是让printLog方法在切入点方法执行之前，是前置通知
     * aop:before表示配置前置通知
     * method属性：用于指定Logger类中哪个方法是前置通知
     * pointcut属性：用于指定切入点表达式，该表达式指的是对业务层中需要增强的方法			

```java
<bean id="logger" class="com.itheima.log.Logger"></bean>
    <aop:config>
        <aop:aspect id="logAdvice" ref="logger">
            <aop:before method="printLog" 
                pointcut="execution(public void com.itheima.service.impl.*.*(..))"></aop:before>
        </aop:aspect>
    </aop:config>
```

* 切入点表达式：execution(表达式)
  * 表达式：访问修饰符 返回值 包名.包名...类名.方法名(参数列表)
    * 标准：public void com.itheima.service.impl.UserServiceImpl.save()
    * 全通配写法：* *..*.*(..)
      * 访问修饰符可以省略
      * 返回值使用通配符 * 
      * 包名使用通配符 *..
      * 类名和方法名使用 *.*
      * 参数列表使用 (..)
      * 实际开发中：切入到业务层实现类的方法   【* com.itheima.service.impl.*.*(..)】
    * 切入点方法可以使用标签<aop:pointcut>，放在标签<aop:aspect>内部，也可以放在标签<aop:aspect>前面

```java
<aop:config>
	<aop:pointcut id="ptl" expression="execution(* com.itheima.service.impl.*.*(..))"/>
        <aop:aspect id="logAdvice" ref="logger">
            <aop:before method="beforePrintLog" pointcut-ref="ptl"></aop:before>
            <aop:after-returning method="afterReturning" pointcut-ref="ptl"></aop:after-returning>
            <aop:after-throwing method="afterThrowing" pointcut-ref="ptl"></aop:after-throwing>
            <aop:after method="after" pointcut-ref="ptl"></aop:after>
            <!-- <aop:pointcut id="ptl" expression="execution(* com.itheima.service.impl.*.*(..))"/> -->
        </aop:aspect>
    </aop:config>
```

环绕通知

```java
    /**
     * 环绕通知
     */
    public Object arroundPrintLog(ProceedingJoinPoint pj) {
        Object value = null;
        try {
            Object[] args = pj.getArgs();
            System.out.println("*****打印日志*****前置通知");
            value = pj.proceed(args);
            System.out.println("*****打印日志*****后置通知");
            return value;
        } catch (Throwable throwable) {
            System.out.println("*****打印日志*****异常通知");
            throw new RuntimeException(throwable);
        } finally {
            System.out.println("*****打印日志*****最终通知");
        }
    }
```

声明式事务配置《xml》

```java
<!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="datasource"></property>
    </bean>
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!--配置事务的属性
                isolation:用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别
                propagation:用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务
                read-only:用于指定事务是否只读。默认值是false，查询设置true。
                timeout:用于指定事务的超时时间。默认值是-1，表示永不超时
                rollback-for:用于指定一个异常，当发生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值，表示任何异常都回滚。
                no-rollback-for:用于指定一个异常，发生该异常时，事务不回滚，产生其他异常时，事务回滚。没有默认值，表示任何异常都回滚。
        -->
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="find" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>

    <!--配置AOP-->
    <aop:config>
        <!--配置切入点表达式-->
        <aop:pointcut id="ptl" expression="execution(* com.itheima.service.impl.*.*(..))"/>
        <!--建立切入点表达式和事务通知的对应关系-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="ptl"></aop:advisor>
    </aop:config>
```

声明式事务配置《注解》

```java
<!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="datasource"></property>
    </bean>
    <!--开启spring对事务注解的支持-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
    <!--在需要增强的方法上配置注解 @Transaction -->
```




