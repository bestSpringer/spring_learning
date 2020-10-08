# swgger2使用

swgger是为了描述一套标准的而且和语言无关的REST API的规范。

#### 引入依赖

maven依赖

```java
<properties>
	<swagger.version>2.9.2</swagger.version>
</properties>
<!--swgger2API-->
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>${swagger.version}</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>${swagger.version}</version>
</dependency>
```

springfox-swgger2是依赖OSA（Open System Adminstrator），意指开源的，开放的运维管理系统。

