# SpringCloud笔记

## 1、前言

```
all in one=====>模块化的开发
	
微服务的四个核心问题：
	这么多服务
	1.客户端怎么访问？
	2.服务之间如何通信？
	3.如何治理？
	4.服务挂了怎么办？
	
解决方案：
	
	1.Spring Cloud NetFlix 一站式解决方案
		API网关：zuul组件
		通信：Feign ---- HttpClient ---- Http通信方式,同步,阻塞
		服务注册和发现：Eureka
		熔断机制：Hystrix

    2. Apache Dubbo Zookeeper：半自动，需要整合别人的，这个方案并不完善
    	API网关：没有,找第三方组件(比如整合zull组件),或者自己实现
    	通信：Dubbo 是一个基于Java的高性能的RPC通信框架(性能比Feign强大)
    	服务注册和发现：Zookeeper
    	熔断机制：没有,需要借助Hystrix
    	
    3. Spring Cloud Alibaba：目前最新的一站式解决方案，可更简单的解决上述4个核心问题
    	Netflix五大神兽：Eureka,Ribbon,Feign,Hystrix,Zuul
			
万变不离其宗4个问题：
	1. API网关
	2. HTTP,RPC通信
	3. 注册和发现
	4. 熔断机制
```



## 2. 微服务概述

#### 2.1 什么是微服务？

微服务是一种架构模式，或者说是一种架构风格，**它将单一的应用程序划分成一组小的服务**，每个服务运行在自己独立的进程内，服务之间互相协调，互相配置，为用户提供最终价值，服务之间采用轻量级的通信机制(**HTTP**)互相沟通，每个服务都围绕着具体的业务进行构建，并且能狗被独立的部署到生产环境中。另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应该根据业务上下文，选择合适的语言，工具(**Maven**)对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务，可以使用不同的语言来编写服务，也可以使用不同的数据存储。

**再来从技术维度角度理解下：**

微服务的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合，每一个微服务提供单个业务功能，拥有自己独立的数据库。

#### 2.3 微服务优缺点

**优点**

- 每个服务足够内聚，足够小，代码容易理解，这样能聚焦一个指定的业务功能或业务需求。
- 耦合度比较低，不会影响其他模块的开发。
- 配置比较简单，基本用注解就能实现，不用使用过多的配置文件。
- 直接写后端的代码，不用关注前端怎么开发，直接写自己的后端代码即可，然后暴露接口，通过组件进行服务通信。
- 微服务能使用不同的语言开发。
- 每个微服务都有自己的存储能力，可以有自己的数据库，也可以有统一的数据库。

**缺点**

- 多服务运维难度，随着服务的增加，运维的压力也在增大；
- 数据一致性问题；
- 系统部署依赖问题；
- 系统集成测试问题；
- 性能和监控问题；

#### 2.4 微服务技术栈有那些？

| **微服务技术条目**                     | 落地技术                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| 服务开发                               | SpringBoot、Spring、SpringMVC等                              |
| 服务配置与管理                         | Netfix公司的Archaius、阿里的Diamond等                        |
| 服务注册与发现                         | Eureka、Zookeeper等                                          |
| 服务调用                               | Rest、RPC、gRPC                                              |
| 服务熔断器                             | Hystrix、Envoy等                                             |
| 负载均衡                               | Ribbon、Nginx等                                              |
| 服务接口调用(客户端调用服务的简化工具) | Fegin等                                                      |
| 消息队列                               | Kafka、RabbitMQ等                                            |
| 服务配置中心管理                       | SpringCloudConfig、Chef等                                    |
| 服务路由(API网关)                      | Zuul等                                                       |
| 服务监控                               | Zabbix、Nagios、Metrics、Specatator等                        |
| 全链路追踪                             | Zipkin、Brave、Dapper等                                      |
| 数据流操作开发包                       | SpringCloud Stream(封装与Redis，Rabbit，Kafka等发送接收消息) |
| 时间消息总栈                           | SpringCloud Bus                                              |
| 服务部署                               | Docker、Kubernetes等                                         |



## 3. SpringCloud入门概述

### 3.1 SpringCloud是什么？

SpringCloud是基于SpringBoot提供的一套微服务解决方案，包括服务注册与发现，配置中心，服务网关，负载均衡，熔断器，全链路监控等组件。

SpringBoot并没有重复造轮子，它只是将目前各家公司开发的比较成熟的服务框架组合起来，通过SpringBoot风格进行再封装，屏蔽掉了复杂的配置和实现原理，**最终给开发者留出了一套简单易懂，易部署和维护的分布式系统开发工具包**

### 3.2 SpringCloud和SpringBoot的关系

- SpringBoot专注于开发单个微服务；
- SpringCloud是关注全局的微服务协调治理框架，它将SpringBoot开发的一个个单体微服务，整合并管理起来，为各个微服务之间提供：服务注册与发现，配置中心，服务网关，负载均衡，熔断器，全链路监控等集成服务。

### 3.3 Dubbo 和 SpringCloud技术选型

1. **Spring Cloud 抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式。**这两种方式各有优劣。虽然从一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生RPC带来的问题。而且REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这个优点在当下强调快速演化的微服务环境下，显得更加合适。
2. 注册中心：dubbo 是zookeeper springcloud是eureka，也可以是zookeeper。
3. 服务网关，dubbo本身没有实现，只能通过其他第三方技术整合，springcloud有Zuul路由网关。



### 3.4 SpringCloud重要组件

Eureka：服务治理组件，包括服务端的注册中心和客户端的服务发现机制；

Ribbon：负载均衡的服务调用组件，具有多种负载均衡调用策略；

Hystrix：服务容错组件，实现了断路器模式，为依赖服务的出错和延迟提供了容错能力；

Feign：基于Ribbon和Hystrix的声明式服务调用组件；

Zuul：API网关组件，对请求提供路由及过滤功能。



## 4. SpringCloud Rest学习环境搭建：服务提供者

### 4.1 介绍

我们会使用一个Dept部门模块作为Consumer消费者(Client)，通过REST调用Provider提供者(Server)提供的服务。

MicroServiceCloud父工程(Project)下初次带着3个子模块(Module)

- springcloud-api 【entity/接口/公共配置等】
- springcloud-consumer-dept-80 【服务提供者】
- springcloud-provider-dept-8001 【服务消费者】

### 4.3 创建工程

**1、创建父工程**

- 新建父工程项目springcloud，定义POM文件，将后续各个子模块公用的jar包等统一提取出来，类似一个抽象父类。

- **父工程pom.xml**

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>nuc.ss</groupId>
      <artifactId>springcloud</artifactId>
      <version>1.0-SNAPSHOT</version>
      <modules>
  
      </modules>
  
      <!--打包方式 pom-->
      <packaging>pom</packaging>
  
      <properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          <maven.compiler.source>1.8</maven.compiler.source>
          <maven.compiler.target>1.8</maven.compiler.target>
          <junit.version>4.12</junit.version>
          <lombok.version>1.18.12</lombok.version>
          <log4j.version>1.2.17</log4j.version>
      </properties>
  
      <dependencyManagement>
          <dependencies>
              <!--springcloud的依赖-->
              <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-dependencies</artifactId>
                  <version>Hoxton.SR8</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
              <!--springboot的依赖-->
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-dependencies</artifactId>
                  <version>2.3.1.RELEASE</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
              <!--数据库-->
              <dependency>
                  <groupId>mysql</groupId>
                  <artifactId>mysql-connector-java</artifactId>
                  <version>8.0.20</version>
              </dependency>
              <dependency>
                  <groupId>com.alibaba</groupId>
                  <artifactId>druid</artifactId>
                  <version>1.0.9</version>
              </dependency>
              <!--springboot启动器-->
              <dependency>
                  <groupId>org.mybatis.spring.boot</groupId>
                  <artifactId>mybatis-spring-boot-starter</artifactId>
                  <version>2.1.3</version>
              </dependency>
  
              <!--日志和测试-->
              <dependency>
                  <groupId>ch.qos.logback</groupId>
                  <artifactId>logback-core</artifactId>
                  <version>1.2.3</version>
              </dependency>
              <dependency>
                  <groupId>junit</groupId>
                  <artifactId>junit</artifactId>
                  <version>${junit.version}</version>
              </dependency>
              <dependency>
                  <groupId>log4j</groupId>
                  <artifactId>log4j</artifactId>
                  <version>${log4j.version}</version>
              </dependency>
              <!--lombok-->
              <dependency>
                  <groupId>org.projectlombok</groupId>
                  <artifactId>lombok</artifactId>
                  <version>${lombok.version}</version>
              </dependency>
          </dependencies>
      </dependencyManagement>
      
      <build>
          <resources>
              <resource>
                  <directory>src/main/java</directory>
                  <includes>
                      <include>**/*.yml</include>
                      <include>**/*.properties</include>
                      <include>**/*.xml</include>
                  </includes>
                  <filtering>false</filtering>
              </resource>
              <resource>
                  <directory>src/main/resources</directory>
                  <includes>
                      <include>**/*.yml</include>
                      <include>**/*.properties</include>
                      <include>**/*.xml</include>
                  </includes>
                  <filtering>false</filtering>
              </resource>
          </resources>
      </build>
  </project>
  ```



**每个模块的创建流程**

1. 导入依赖
2. 编写配置文件
3. 开启这个功能 @EnableXXXX
4. 配置类



**2、创建子模块springcloud-api**

**pom配置：**

- ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <parent>
          <artifactId>springcloud</artifactId>
          <groupId>nuc.ss</groupId>
          <version>1.0-SNAPSHOT</version>
      </parent>
      <modelVersion>4.0.0</modelVersion>
  
      <artifactId>sprintcloud-api</artifactId>
  
      <!--当前的module自己需要的依赖，如果父依赖中已经配置了版本，这里就不用写了-->
      <dependencies>
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
          </dependency>
      </dependencies>
  </project>
  ```
  
- **数据库的创建**

  ```mysql
  CREATE DATABASE `db01`
  USE `db01`;
  DROP TABLE IF EXISTS `dept`;
  
  CREATE TABLE `dept` (
    `deptno` BIGINT(20) NOT NULL AUTO_INCREMENT,
    `dname` VARCHAR(60) DEFAULT NULL,
    `db_source` VARCHAR(60) DEFAULT NULL,
    PRIMARY KEY (`deptno`)
  ) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='部门表';
  
  
  INSERT  INTO `dept`(`dname`,`db_source`) 
  VALUES ('开发部',DATABASE()),('人事部',DATABASE()),('财务部',DATABASE()),('市场部',DATABASE()),('运维部',DATABASE());
  ```

- **实体类的编写**

    ```java
    package nuc.ss.springcloud.pojo;
    
    import lombok.Data;
    import lombok.NoArgsConstructor;
    import lombok.experimental.Accessors;
    import java.io.Serializable;
    
    @Data
    @NoArgsConstructor
    @Accessors(chain = true)
    public class Dept implements Serializable {
    
        private long deptno;//主键
        private String dname;
    
        //这个数据存在哪个数据库
        private String db_source;
    
        public Dept(String dname) {
            this.dname = dname;
        }
    
        /*
        * 链式写法：
        * Dept dept = new Dept();
        *
        * dept.setDeptNo(11).setDname('ssss').setDb_source('db01')
        * */
    }
    ```

**3、子模块springcloud-provider-dept-8081服务的提供者的编写**

​	<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200924195248477.png" alt="image-20200924195248477" style="zoom:50%;" />

**pom配置：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud</artifactId>
        <groupId>nuc.ss</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springcloud-provider-dept-8081</artifactId>

    <dependencies>
        <!--我们需要拿到实体类，所以要配置api module-->
        <dependency>
            <groupId>nuc.ss</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>

        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--jetty-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>

        <!--热部署工具-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

**application.yml的配置**

```yaml
server:
  port: 8081

# mybatis的配置
mybatis:
  type-aliases-package: nuc.ss.springcloud.pojo
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml

# spring的配置
spring:
  application:
    name: springcloud-provider-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource #数据库
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql//localhost:3306/db01?useUnicode=true&characterEncoding=utf-8
    username: root
    password: admin
```

**mybatis-config.xml的配置**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
</configuration>
```

**DeptMapper.xml的编写**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="nuc.ss.springcloud.dao.DeptDao">

    <insert id="addDept" parameterType="Dept">
        insert into dept (dname,db_source)
        values (#{dname},DATABASE())
    </insert>

    <select id="queryById" resultType="Dept" parameterType="Long">
        select * from dept where deptno = #{id}
    </select>
    
    <select id="queryAll" resultType="Dept">
        select * from dept
    </select>

</mapper>
```

**接口DeptController的编写**

```java
@RestController
public class DeptController {
    @Autowired
    DeptService deptService;

    @PostMapping("/dept/add")
    public boolean addDept(Dept dept) {
        return deptService.addDept(dept);
    }

    @GetMapping("/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id) {
        return deptService.queryById(id);
    }

    @GetMapping("/dept/list")
    public List<Dept> queryAll() {
        return deptService.queryAll();
    }
}
```

**整体目录结构**

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200925201520700.png" alt="image-20200925201520700" style="zoom:50%;" />

**4、子模块springcloud-consumer-dept-80的编写**

- pom

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <parent>
          <artifactId>springcloud</artifactId>
          <groupId>nuc.ss</groupId>
          <version>1.0-SNAPSHOT</version>
      </parent>
      <modelVersion>4.0.0</modelVersion>
  
      <artifactId>springcloud-consumer-dept-80</artifactId>
  
      <dependencies>
          <dependency>
              <groupId>nuc.ss</groupId>
              <artifactId>springcloud-api</artifactId>
              <version>1.0-SNAPSHOT</version>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
      </dependencies>
  </project>
  ```

- application.yml

  ```yaml
  server:
    port: 80
  ```

- **将RestTemplate注册到spring中：**ConfigBean.java

  ```java
  @Configuration
  public class ConfigBean {
  
      @Bean
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }
  ```

- DeptConsumerController.java

  ```java
  @RestController
  public class DeptConsumerController {
      
      // 消费者不应该有service层
      // RestTemplate供我们直接调用就可以了，将它注册到spring中
      @Autowired
      private RestTemplate restTemplate;//提供多种访问Http的方法
  
      private static final String REST_URL_PREFIX = "http://localhost:8081";
  
      @RequestMapping("/consumer/dept/add")
      public boolean add(Dept dept) {
          // (url,实体:Map, Class<T> responseType) 
          return 
             restTemplate.postForObject(REST_URL_PREFIX+"/dept/add",dept,Boolean.class);
      }
  
      @RequestMapping("/consumer/dept/get/{id}")
      public Dept get(@PathVariable("id") Long id) {
          return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id,Dept.class);
      }
  
      @RequestMapping("/consumer/dept/list")
      public List<Dept> list() {
          return restTemplate.getForObject(REST_URL_PREFIX+"/dept/list",List.class);
      }
  }
  ```

- **启动服务: **DeptConsumer_80

  ```java
  @SpringBootApplication
  public class DeptConsumer_80 {
      public static void main(String[] args) {
          SpringApplication.run(DeptConsumer_80.class,args);
      }
  }
  ```

  

## 5、Eureka服务注册与发现

### 5.1 什么是Eureka

Eureka是基于REST的服务，用于定位服务，以实现云端中间件层服务发现和故障转移。
服务注册与发现对于微服务来说是非常重要，有了服务注册与发现，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了，功能类似于Zookeeper。

### 5.2 原理理解

- **Eureka基本的架构**

  - Springcloud 封装了Netflix公司开发的Eureka模块来实现服务注册与发现 。

  - Eureka采用了C-S的架构设计，**EurekaServer**作为服务注册功能的服务器，是**服务注册中心。**各个节点启动后，会在EurekaServer中进行注册，这样Eureka Server中的服务注册表中将会储存所有可用服务节点的信息。

  - **系统中的其他微服务，使用Eureka的客户端连接到EurekaServer并维持心跳连接。**这样系统的维护人员就可以通过EurekaServer来监控系统中各个微服务是否正常运行，Springcloud 的一些其他模块 (比如Zuul) 就可以通过EurekaServer来发现系统中的其他微服务，并执行相关的逻辑。

    ![image-20200926091541672](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200926091541672.png)

- **三大角色**

  - Eureka Server：服务的注册与发现
  - Service Provider：服务生产方，将自身服务注册到Eureka中，从而使服务消费方能够找到
  - Service Consumer：服务消费方，从Eureka中获取注册服务列表，从而找到消费服务

### 5.3 构建步骤

**1. eureka-server**

1. springcloud-eureka-7001 模块建立

2. pom.xml 配置

   ```xml
   <dependencies>
       <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-eureka-server</artifactId>
           <version>1.4.6.RELEASE</version>
       </dependency>
   
       <!--热部署-->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-devtools</artifactId>
       </dependency>
   </dependencies>
   ```

3. application.yml

   ```yaml
   server:
     port: 7001
   
   # Eureka配置
   eureka:
     instance:
       hostname: localhost # Eureka Service的名字
     client:
       register-with-eureka: false # 是否向Eureka中心注册自己
       fetch-registry: false # 如果为false，则表示自己为注册中心
       # Eureka Service的访问路径
       service-url:
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

4. 主启动类EurekaServer_7001.java

   ```java
   @SpringBootApplication
   @EnableEurekaServer
   public class EurekaServer_7001 {
       public static void main(String[] args) {
           SpringApplication.run(EurekaServer_7001.class,args);
       }
   }
   ```

5. 启动成功后访问 http://localhost:7001/ 得到以下页面

   ![image-20200926110158171](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200926110158171.png)

**2. eureka-client** 服务提供者

- **调整之前创建的springlouc-provider-dept-8081**

  - **导入Eureka依赖**

    ```xml
    <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>
    ```

  - application.yml中新增Eureka配置

    ```yaml
    # Eureka的配置
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:7001/eureka/
    ```

  - 为主启动类添加@EnableEurekaClient注解

    ```java
    //启动类
    @SpringBootApplication
    @EnableEurekaClient //在服务启动后自动注册到Eureka中
    public class DeptProvider_8081 {
        public static void main(String[] args) {
            SpringApplication.run(DeptProvider_8081.class,args);
        }
    }
    ```

  - 先启动7001服务端后启动8001客户端进行测试，然后访问监控页http://localhost:7001/ 

    ![image-20200926120411242](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200926120411242.png)

  - **修改Eureka Client上的默认描述信息**

    ```yaml
    # Eureka的配置
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:7001/eureka/
      instance:
        instance-id: springcloud-provider-dept-8081 #修改Eureka上的默认描述信息
    ```

    **结果如图：**

    ![image-20200926121313373](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200926121313373.png)




**3. EureKa自我保护机制：**某时刻某一个微服务不可用，eureka不会立即清理，依旧会对该微服务的信息进行保存

- 当eureka server在一定时间内没有收到实例的心跳，便会把该实例从注册表中删除（**默认是90秒**），但是，如果短时间内丢失大量的心跳，便会触发eureka server的自我保护机制。比如在开发测试时，需要频繁地重启微服务实例，但是我们很少会把eureka server一起重启（因为在开发过程中不会修改eureka注册中心），**当一分钟内收到的心跳数大量减少时，会触发该保护机制**。eureka认为虽然收不到实例的心跳，但它认为实例还是健康的，eureka会保护这些实例，不会把它们从注册表中删掉。
- 该保护机制的目的是避免网络连接故障，在发生网络故障时，微服务和注册中心之间无法正常通信，但服务本身是健康的，不应该注销该服务，如果eureka因网络故障而把微服务误删了，那即使网络恢复了，该微服务也不会重新注册到eureka server了，因为只有在微服务启动的时候才会发起注册请求，后面只会发送心跳和服务列表请求，这样的话，该实例虽然是运行着，但永远不会被其它服务所感知。

**4. 注册进来的微服务，获取一些消息（团队开发会用到）**

DiscoveryClient的作用：可以从注册中心中根据服务别名获取注册的服务器信息。

- 启动类添加注解`@EnableDiscoveryClient`（服务发现）

  ```java
  //启动类
  @SpringBootApplication
  @EnableEurekaClient
  @EnableDiscoveryClient
  public class DeptProvider_8081 {
      public static void main(String[] args) {
          SpringApplication.run(DeptProvider_8081.class,args);
      }
  }
  ```

- DeptController.java新增方法

  ```java
  //获取一些配置的信息，得到具体的微服务！
  @Autowired
  private DiscoveryClient client;
  
   //获取一些注册进来的微服务的消息
   @GetMapping("/dept/discovery")
   public Object discovery() {
       //获取微服务列表的清单
       List<String> services = client.getServices();
       System.out.println("discovery=>services:" + services);
       //得到一个具体的微服务信息,通过具体的微服务id，applicaioinName；
       List<ServiceInstance> instances = client.getInstances("SPRINGCLOUD-PROVIDER-DEPT");
       for (ServiceInstance instance : instances) {
           System.out.println(
                   instance.getHost() + "\t" + // 主机名称
                           instance.getPort() + "\t" + // 端口号
                           instance.getUri() + "\t" + // uri
                           instance.getServiceId() // 服务id
           );
       }
       return this.client;
   }
  
  ```

- 结果

  <img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927155502206.png" alt="image-20200927155502206" style="zoom:50%;" />



### 5.4 Eureka：集群环境配置

**1.初始化**

- 新建springcloud-eureka-7002、springcloud-eureka-7003 模块

- 为pom.xml添加依赖 (与springcloud-eureka-7001相同)

- application.yml配置(与springcloud-eureka-7001相同)，端口号分别换成7002和7003

- 主启动类(与springcloud-eureka-7001相同)

**2.集群成员相互关联**

- 配置一些自定义本机名字，找到本机hosts文件并打开

- 在hosts文件最后加上，要访问的本机名称，默认是localhost

  <img src="C:\Users\黄睿楠\AppData\Roaming\Typora\typora-user-images\image-20220417170025257.png" alt="image-20220417170025257" style="zoom:50%;" />

- 修改Eureka Service的名字

  <img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927164209557.png" alt="image-20200927164209557" style="zoom:50%;" />

- 使springcloud-eureka-7001关联springcloud-eureka-7002、springcloud-eureka-7003

  以7001为例：

  ```yaml
  server:
    port: 7001
  
  # Eureka配置
  eureka:
    instance:
      hostname: eureka7001.com # Eureka服务端的名字
    client:
      register-with-eureka: false
      fetch-registry: false 
      service-url:  #监控页面
        # 集群（关联）：7001关联7002、7003
        defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  
  ```
  
- 通过springcloud-provider-dept-8081下的yml配置文件，修改**Eureka配置：配置服务注册中心地址**

  ```yaml
  # Eureka的配置
  eureka:
    client:
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
    instance:
      instance-id: springcloud-provider-dept-8081 #修改Eureka上的默认描述信息
  ```

- 这样模拟集群就搭建好了，就可以把一个项目挂载到三个服务器上了

  ![image-20200927170039729](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927170039729.png)

### 5.5 Eureka和Zookeeper区别

**1. CAP是什么?**

- C (Consistency) 强一致性
- A (Availability) 可用性
- P (Partition tolerance) 分区容错性

**2. <font color=red>CAP理论的核心</font>**:一个分布式系统不可能同时满足一致性，可用性和分区容错性这三个需求

**3. Eureka比Zookeeper好在哪里？**

由于分区容错性P再分布式系统中是必须要保证的，因此我们只能再A和C之间进行权衡。

- Zookeeper 保证的是CP
- Eureka 保证的是AP

**Zookeeper保证的是CP**

zookeeper会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。但选举leader的时间太长，30-120s，且选举期间整个zookeeper集群是不可用的，这就导致在选举期间注册服务瘫痪。虽然服务最终能够恢复，但是，漫长的选举过程会导致注册长期不可用。

**Eureka保证的是AP**

Eureka在设计时就优先保证可用性。**Eureka各个节点都是平等的**，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。Eureka的客户端在向某个Eureka注册时，如果发现连接失败，则会自动切换至其他节点，只要有一台Eureka还在，就能保住注册服务的可用性，只不过查到的信息可能不是最新的，除此之外，Eureka还有自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时：

- Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
- Eureka仍然能够接受新服务的注册和查询请求，但是不会同步到其他节点上 (即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其他节点中

<font color=red>因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪</font>



## 6. Ribbon：负载均衡(基于客户端)

### 6.1 负载均衡以及Ribbon

**Ribbon是什么？**

- Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法

  Ribbon客户端组件提供一系列完善的配置项，如连接超时，重试等。简单的说，就是在配置文件中列出后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随即连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。
- 负载均衡简单分类：
  - 集中式LB：即在服务的提供方和消费方之间使用独立的LB设施，如Nginx，由该设施负责把访问请求通过某种策略转发至服务的提供方。
  - 进程式LB：将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选出一个合适的服务器。
- <font color=red>Ribbon 属于进程内LB</font>

### Ribbon底层实现原理

Ribbon使用discoveryClient从注册中心读取目标服务信息，对同一接口请求进行计数，使用%取余算法获取目标服务集群索引，返回获取到的目标服务信息。

### Nginx与Ribbon的区别

Nginx是反向代理同时可以实现负载均衡，nginx拦截客户端请求采用负载均衡策略根据upstream配置进行转发，相当于请求通过nginx服务器进行转发。Ribbon是客户端负载均衡，从注册中心读取目标服务器信息，然后客户端采用轮询策略对服务直接访问，全程在客户端操作。

### 6.2 集成Ribbon

- **springcloud-consumer-dept-80**向pom.xml中添加Ribbon和Eureka依赖

  ```xml
  <!-- Ribbon -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-ribbon</artifactId>
      <version>1.4.7.RELEASE</version>
  </dependency>
  
  <!--eureka-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka</artifactId>
      <version>1.4.7.RELEASE</version>
  </dependency>
  ```

- 在application.yml文件中配置Eureka

  ```yaml
  # Eureka配置
  eureka:
    client:
      register-with-eureka: false
      service-url: # 从三个注册中心中随机取一个去访问
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  ```

- 主启动类加上@EnableEurekaClient注解，开启Eureka

  （服务的提供方和消费方都需要加@EnableEurekaClient注解）

  ```java
  @SpringBootApplication
  @EnableEurekaClient //开启Eureka客户端
  public class DeptConsumer_80 {
      public static void main(String[] args) {
          SpringApplication.run(DeptConsumer_80.class,args);
      }
  }
  ```

- 自定义Spring配置类：ConfigBean.java 配置负载均衡实现RestTemplate

  ```java
  @Configuration
  public class ConfigBean {   //Cofiguration -- spring applicationContext.xml
  
      @LoadBalanced //配置负载均衡实现RestTemplate
      @Bean
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }   
  }
  ```

### 6.3 使用Ribbon实现负载均衡

**1.实现负载均衡**

- 流程图：

  ![image-20200927193721521](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927193721521.png)

- 创建db02和db03数据库(与db01一样)

- 新建两个服务提供者Moudle：springcloud-provider-dept-8082、springcloud-provider-dept-8083

- 参照springcloud-provider-dept-8081 依次为另外两个Moudle添加pom.xml依赖 、resourece下的mybatis和application.yml配置，Java代码

  <font color=red>三个服务（spring.application.name）的名称必须一致</font>

- 启动所有服务测试，访问http://localhost/consumer/dept/list 这时候随机访问的是服务提供者8081

  <img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927193308957.png" alt="image-20200927193308957" style="zoom:50%;" />

- 再次访问http://localhost/consumer/dept/list这时候随机的是服务提供者8083

  <img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927193329021.png" alt="image-20200927193329021" style="zoom:50%;" />

- 再次访问http://localhost/consumer/dept/list这时候随机的是服务提供者8082

  <img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927193346522.png" alt="image-20200927193346522" style="zoom:50%;" />



**2.如何切换或者自定义规则呢？**

- 在springcloud-provider-dept-80模块下的ConfigBean中进行配置，切换使用不同的规则

  ```java
  @Configuration
  public class ConfigBean {//@Configuration -- spring  applicationContext.xml
  
      /**
       * IRule:
       * RoundRobinRule 轮询
       * RandomRule 随机
       * AvailabilityFilteringRule ： 会先过滤掉跳闸的服务，对剩下的进行轮询
       * RetryRule ： 会先按照轮询获取服务，如果服务获取失败，则会在指定的时间内进行重试
       */
      @LoadBalanced
      @Bean
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  
      @Bean
      public IRule myRule(){
          return new RandomRule();//使用随机规则
      }
  }
  
  ```

- 也可以自定义规则，在myRule包下自定义一个配置类MyRule.java，注意：**<font color=red>该包不要和主启动类所在的包同级，要跟启动类所在包同级</font>**：
  <img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002124601956.png" alt="image-20201002124601956" style="zoom:50%;" />

- MyRule.java

  ```java
  @Configuration
  public class MyRule {
  
      @Bean
      public IRule hrnMyRule() {
          return new MyRandomRule();//默认是轮循，现在我们自定义
      }
  }
  ```

- 主启动类开启负载均衡并指定自定义的MyRule配置类

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  //在微服务启动的时候就能加载自定义的Ribbon类(自定义的规则会覆盖原有默认的规则)
  @RibbonClient(name = "SPRINGCLOUD-PROVIDER-DEPT",configuration = MyRule.class)//开启负载均衡,并指定自定义的规则
  public class DeptConsumer_80 {
      public static void main(String[] args) {
          SpringApplication.run(DeptConsumer_80.class,args);
      }
  }
  ```
  
- 自定义的规则(这里我们参考Ribbon中默认的规则代码自己稍微改动)：MyRandomRule.java

  ```java
  package nuc.ss.myrule;
  
  import com.netflix.client.config.IClientConfig;
  import com.netflix.loadbalancer.AbstractLoadBalancerRule;
  import com.netflix.loadbalancer.ILoadBalancer;
  import com.netflix.loadbalancer.Server;
  
  import java.util.List;
  import java.util.concurrent.ThreadLocalRandom;
  
  public class MyRandomRule extends AbstractLoadBalancerRule {
  
      /**
       * 每个服务访问5次，则换下一个服务(总共3个服务)
       * total=0,默认=0,如果=5,指向下一个服务节点
       * index=0,默认=0,如果total=5,index+1
       */
  
      private int total = 0;          //被调用的次数
      private int currentIndex = 0;   //当前是谁在提供服务
  
      public Server choose(ILoadBalancer lb, Object key) {
          if (lb == null) {
              return null;
          }
          Server server = null;
  
          while (server == null) {
              if (Thread.interrupted()) {
                  return null;
              }
              List<Server> upList = lb.getReachableServers(); //获得当前活着的服务
              List<Server> allList = lb.getAllServers(); //获取所有的服务
  
              int serverCount = allList.size();
              if (serverCount == 0) {
                  return null;
              }
  
              //int index = chooseRandomInt(serverCount);//生成区间随机数
              //server = upList.get(index);//从或活着的服务中,随机获取一个
  
              //=====================自定义代码=========================
  
              if (total < 5) {
                  server = upList.get(currentIndex);
                  total++;
              } else {
                  total = 0;
                  currentIndex++;
                  if (currentIndex >= upList.size()) {
                      currentIndex = 0;
                  }
                  //server = upList.get(currentIndex);//从活着的服务中,获取指定的服务来进行操作
              }
  
              //======================================================
  
              if (server == null) {
                  Thread.yield();
                  continue;
              }
  
              if (server.isAlive()) {
                  return (server);
              }
              server = null;
              Thread.yield();
          }
  
          return server;
  
      }
  
      protected int chooseRandomInt(int serverCount) {
          return ThreadLocalRandom.current().nextInt(serverCount);
      }
  
  	@Override
  	public Server choose(Object key) {
  		return choose(getLoadBalancer(), key);
  	}
  
  	@Override
  	public void initWithNiwsConfig(IClientConfig clientConfig) {
  		// TODO Auto-generated method stub
  		
  	}
  }
  ```




## 7.Feign：负载均衡(基于服务端)

### 7.1 Feign简介

Feign是声明式Web Service客户端，它让微服务之间的调用变得更简单，类似controller调用service。SpringCloud集成了Ribbon和Eureka，可以使用Feigin提供负载均衡的http客户端

**只需要创建一个接口，然后添加注解即可~**

Feign，主要是社区版，大家都习惯面向接口编程。这个是很多开发人员的规范。调用微服务访问两种方法

1. 微服务名字 【ribbon】
2. 接口和注解 【feign】

**Feign能干什么？**

- Feign旨在使编写Java Http客户端变得更容易
- 前面在使用Ribbon + RestTemplate时，利用RestTemplate对Http请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一个客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步的封装，由他来帮助我们定义和实现依赖服务接口的定义，<font color=red>在Feign的实现下，我们只需要创建一个接口并使用注解的方式来配置它 (类似以前Dao接口上标注Mapper注解，现在是一个微服务接口上面标注一个Feign注解即可</font>)，即可完成对服务提供方的接口绑定，简化了使用Spring Cloud Ribbon 时，自动封装服务调用客户端的开发量。

**Feign默认集成了Ribbon**

- 利用Ribbon维护了MicroServiceCloud-Dept的服务列表信息，并且通过轮询实现了客户端的负载均衡，而与Ribbon不同的是，通过Feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。

### 7.2 Feign的使用步骤

1. 改造springcloud-api模块

   pom.xml添加feign依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-feign</artifactId>
       <version>1.4.7.RELEASE</version>
   </dependency>
   ```

   新建service层，并新建DeptClientService.java接口，

   ```java
   @Service
   //@FeignClient:微服务客户端注解,value:指定微服务的名字,这样就可以使Feign客户端直接找到对应的微服务
   @FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT")
   public interface DeptClientService {
   
       @GetMapping("/dept/get/{id}")
       Dept queryById(@PathVariable("id") Long id);
   
       @GetMapping("/dept/list")
       List<Dept> queryAll();
   
       @PostMapping("/dept/add")
       boolean addDept(Dept dept);
   }
   ```

   

2. 创建springcloud-consumer-dept-feign模块

   拷贝springcloud-consumer-dept-80模块下的pom.xml，resource，以及java代码到springcloud-consumer-feign模块，并添加feign依赖。

   ```xml
   <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-feign -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-feign</artifactId>
       <version>1.4.7.RELEASE</version>
   </dependency>
   ```

   通过Feign实现DeptConsumerController.java

   ```java
   @RestController
   public class DeptConsumerController {
   
       @Autowired
       private DeptClientService deptClientService = null;
   
       @RequestMapping("/consumer/dept/add")
       public boolean add(Dept dept) {
           return this.deptClientService.addDept(dept);
       }
   
       @RequestMapping("/consumer/dept/get/{id}")
       public Dept get(@PathVariable("id") Long id) {
          return this.deptClientService.queryById(id);
       }
   
       @RequestMapping("/consumer/dept/list")
       public List<Dept> list() {
           return this.deptClientService.queryAll();
       }
   }
   
   ```

   主配置类

   ```java
   @SpringBootApplication
   @EnableEurekaClient //开启Eureka 客户端
   @EnableFeignClients(basePackages = {"nuc.ss.springcloud"})
   public class FeignDeptConsumer_80 {
       public static void main(String[] args) {
           SpringApplication.run(FeignDeptConsumer_80.class,args);
       }
   }
   ```

3. 结果

   ![image-20201002135742053](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002135742053.png)

### 7.3 Feign和Ribbon如何选择？

**根据个人习惯而定，如果喜欢REST风格使用Ribbon；如果喜欢社区版的面向接口风格使用Feign.**



## 8. Hystrix：服务熔断

### 8.1 服务雪崩

 多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的“扇出”，如果扇出的链路上**某个微服务的调用响应时间过长，或者不可用**，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃。

 “断路器”是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控 ，**向调用方返回一个服务预期的，可处理的备选响应 (FallBack) ，而不是长时间的等待或者抛出调用方法无法处理的异常，这样就可以保证服务调用方的线程不会被长时间、不必要的占用**，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

### 8.2 什么是Hystrix？

Hystrix是一个用于处理分布式系统的延迟和容错的开源库。在分布式系统里，许多依赖不可避免的会调用失败，比如超时，异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整个体系服务失败，避免级联故障，提高分布式系统的可用性。

Hystrix有四种防雪崩方式:

- 服务熔断：接口调用失败就会进入调用接口提前定义好的一个熔断的方法，返回错误信息
- 服务降级：接口调用失败就调用本地的方法返回一个空
- 服务隔离：Hystrix为隔离的服务开启一个独立的线程池，这样在高并发的情况下不会影响其他服务。
- 服务监控：在服务发生调用时,会将每秒请求数、成功请求数等运行指标记录下来。

### 8.4、服务熔断

#### **8.4.1、什么是服务熔断**

服务熔断是应对雪崩效应的一种微服务链路保护机制。当调用链路的某个微服务不可用或者响应时间太长时，会进行服务熔断，不再有该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务调用响应正常后，恢复调用链路。熔断机制的注解是：**<font color=red>@HystrixCommand</font>** 。

服务熔断解决如下问题： 

- 当所依赖的对象不稳定时，能够起到快速失败的目的
- 快速失败后，能够根据一定的算法动态试探所依赖对象是否恢复

#### 8.4.2、入门案例

新建springcloud-provider-dept-hystrix-8081模块，并拷贝springcloud-provider-dept–8081内的pom.xml、resource和Java代码进行初始化并调整。

**导入hystrix依赖**

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-hystrix -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

**调整yml配置文件**

```yaml
# Eureka的配置
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: springcloud-provider-dept-hystrix-8081 #修改Eureka上的默认描述信息
    prefer-ip-address: true #改为true后默认显示的是ip地址而不再是localhost
```

prefer-ip-address: true：

![image-20201002180111151](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002180111151.png)

**修改controller**

```java
@RestController
public class DeptController {
    @Autowired
    private DeptService deptService;

    @Autowired
    private DiscoveryClient client;

    @GetMapping("/dept/get/{id}")
    @HystrixCommand(fallbackMethod = "hystrixGet")//如果出现异常,走后面的hystrixGet代码
    public Dept get(@PathVariable("id") Long id) {
        Dept dept = deptService.queryById(id);
        if (dept==null){
            throw new RuntimeException("这个id=>"+id+",不存在该用户，或信息无法找到");
        }
        return dept;
    }

    //根据id查询备选方案(熔断)
    public Dept hystrixGet(@PathVariable("id") Long id){

        return new Dept().setDeptno(id)
                .setDname("这个id=>"+id+",没有对应的信息,null---@Hystrix")
                .setDb_source("在MySQL中没有这个数据库");
    }
}
```

**为主启动类添加对熔断的支持注解@EnableCircuitBreaker**

```java
@SpringBootApplication
@EnableEurekaClient //在服务启动后自动注册到Eureka中
@EnableDiscoveryClient //服务发现
@EnableCircuitBreaker//添加对熔断的支持
public class DeptProviderHystrix_8081 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProviderHystrix_8081.class,args);
    }
}
```

**测试**：

使用熔断后，当访问一个存在的id时，前台页展示数据如下

![image-20201002175040487](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002175040487.png)

使用熔断后，当访问一个不存在的id时，前台页展示数据如下

![image-20201002175118082](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002175118082.png)

不用熔断的springcloud-provider-dept–8081模块访问相同地址会出现下面状况

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002175735407.png" alt="image-20201002175735407" style="zoom:50%;" />





### 8.5 服务降级

#### 什么是服务降级

​         服务降级是指 当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心业务正常运作或高效运作。
  资源有限，而请求是无限的。如果在并发高峰期，不做服务降级处理，一方面肯定会影响整体服务的性能，严重的话可能会导致宕机某些重要的服务不可用。所以，一般在高峰期，为了保证核心功能服务的可用性，都要对某些服务降级处理。比如当双11活动时，把交易无关的服务统统降级。
  降级的方式可以根据业务来，可以延迟服务，比如延迟给用户增加积分，只是放到一个缓存中，等服务平稳之后再执行 ；或者在粒度范围内关闭服务，比如关闭相关文章的推荐。

#### 自动降级分类

1）超时降级：主要配置好超时时间和超时重试次数和机制，并使用异步机制探测回复情况

2）失败次数降级：主要是一些不稳定的api，当失败调用次数达到一定阀值自动降级，同样要使用异步机制探测回复情况

3）故障降级：比如要调用的远程服务挂掉了（网络故障、DNS故障、http服务返回错误的状态码、rpc服务抛出异常），则可以直接降级。降级后的处理方案有：默认值（比如库存服务挂了，返回默认现货）、兜底数据（比如广告挂了，返回提前准备好的一些静态页面）、缓存（之前暂存的一些缓存数据）

4）限流降级：秒杀或者抢购一些限购商品时，此时可能会因为访问量太大而导致系统崩溃，此时会使用限流来进行限制访问量，当达到限流阈值，后续请求会被降级；降级后的处理方案可以是：排队页面（将用户导流到排队页面等一会重试）、无货（直接告知用户没货了）、错误页（如活动太火爆了，稍后重试）。

#### 入门案例

在springcloud-api模块下的service包中，新建降级配置类DeptClientServiceFallBackFactory.java

```java
@Component
public class DeptClientServiceFallBackFactory implements FallbackFactory {

    @Override
    public Object create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept queryById(Long id) {
                return new Dept()
                        .setDeptno(id)
                        .setDname("id=>" + id + "没有对应的信息，客户端提供了降级的信息，这个服务现在已经被关闭")
                        .setDb_source("没有数据");
            }

            @Override
            public List<Dept> queryAll() {
                return null;
            }

            @Override
            public boolean addDept(Dept dept) {
                return false;
            }
        };
    }
}
```

在DeptClientService中指定降级配置类DeptClientServiceFallBackFactory

```java
@Service
//@FeignClient:微服务客户端注解,value:指定微服务的名字,这样就可以使Feign客户端直接找到对应的微服务
@FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT",fallbackFactory = DeptClientServiceFallBackFactory.class)
public interface DeptClientService {

    @GetMapping("/dept/get/{id}")
    Dept queryById(@PathVariable("id") Long id);

    @GetMapping("/dept/list")
    List<Dept> queryAll();

    @PostMapping("/dept/add")
    boolean addDept(Dept dept);
}
```

在springcloud-consumer-dept-feign模块中开启降级

```yaml
# 开启降级feign.hystrix
feign:
  hystrix:
    enabled: true
```

**测试**

正常访问

![image-20201002184343632](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002184343632.png)

关掉服务DeptProvider_8081继续访问

![image-20201002183509891](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002183509891.png)



熔断：依赖的下游服务故障触发熔断，避免引发本系统崩溃；系统自动执行和恢复；

降级：服务分优先级，牺牲非核心服务，保证核心服务稳定；从整体负荷考虑；

限流：限制并发的请求访问量，超过阈值则拒绝。


### 8.7 Dashboard 流监控

新建springcloud-consumer-hystrix-dashboard模块

**添加依赖**

```xml
<dependencies>

    <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-hystrix -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>

    <!-- Ribbon -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>

    <!--eureka-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>nuc.ss</groupId>
        <artifactId>springcloud-api</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**主启动类**

```java
@SpringBootApplication
@EnableHystrixDashboard //开启
public class DeptConsumerDashboard_9001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerDashboard_9001.class,args);
    }
}

```

启动应用程序，访问：localhost:9001/hystrix

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002191522305.png" alt="image-20201002191522305" style="zoom:50%;" />

**服务端8081是否有监控应用程依赖，没有添加**

```xml
<!--actuator完善监控信息-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

给springcloud-provider-dept-hystrix-8081模块下的主启动类添加如下代码,添加监控

```java
@SpringBootApplication
@EnableEurekaClient //在服务启动后自动注册到Eureka中
@EnableDiscoveryClient //服务发现
@EnableCircuitBreaker//添加对熔断的支持
public class DeptProviderHystrix_8081 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProviderHystrix_8081.class,args);
    }

    //增加一个 Servlet
    @Bean
    public ServletRegistrationBean hystrixMetricsStreamServlet(){
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
        //访问该页面就是监控页面
        registrationBean.addUrlMappings("/actuator/hystrix.stream");
        return registrationBean;
    }
}
```

**<font color=red>注意：先访问localhost:8081/dept/get/1，再访问localhost:8081/actuator/hystrix.stream，不然也会报错</font>**

在springcloud-consumer-hystrix-dashboard中的yml中添加配置（<font color=red>刚开始没加，一直报这个错: Unable to connect to Command Metric Stream</font>）

```yml
hystrix:
  dashboard:
    proxy-stream-allow-list: "*"
```

![image-20201002213325619](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002213325619.png)

运行结果：（注意心跳和圆的大小变化）

![image-20201002205116058](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002205116058.png)

**如何看运行结果**

- 七色

  ![image-20201002205557442](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002205557442.png)

  绿色：成功数

  蓝色：熔断数

  浅绿色：错误请求数

  黄色：超时数

  紫色：线程池拒绝数

  红色：失败/异常数

  Hosts：服务请求频率

  Circuit Closed：断路状态

- 一圈
  实心圆:公有两种含义，他通过颜色的变化代表了实例的健康程度
  它的健康程度从绿色<黄色<橙色<红色  **递减**
  该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，流量越大，该实心圆就
  越大，所以通过该实心圆的展示，就可以在大量的实例中快速发现**故障实例和高压力实例**。
  ![image-20201002205822580](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002205822580.png)
  ![image-20201002205857949](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002205857949.png)

- 一线
  曲线：用来记录2分钟内流量的相对变化，可以通过它来观察到流量的上升和下降趋势
  ![image-20201002210211989](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002210211989.png)



## 9. Zull路由网关

### 概述

**什么是zuul?**

 Zull包含了对请求的**路由**和**过滤**两个最主要功能：

 其中**路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础**，而**过滤器功能则负责对请求的处理过程进行干预，是实现请求校验，服务聚合等功能的基础**。Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，从Eureka中获得其他服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

网关和过滤器的区别：网关是对所有服务的请求进行分析过滤，过滤器是对单个服务而言。

### Zuul与Nginx有什么区别？

Zuul是java语言实现的，主要为java服务提供网关服务，尤其在微服务架构中可以更加灵活的对网关进行操作。Nginx是使用C语言实现，性能高于Zuul，但是实现自定义操作需要熟悉lua语言，对程序员要求较高，可以使用Nginx做Zuul集群。

### 入门案例

**新建springcloud-zuul模块，并导入依赖**

```xml
<dependencies>
        <!--zuul-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
        <!-- Hystrix依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>

        <!-- Ribbon -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>

        <!--eureka-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>nuc.ss</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

**application.yml**

```yaml
server:
  port: 9527
spring:
  application:
    name: springcloud-zuul
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: zuul9527.com
    prefer-ip-address: true

info:
  app.name: springcloud
  company.name: blog.kuangstudy.com
```

启动如下图三个服务（先去host文件里面添加www.kuangstudy.com的服务）

![image-20201002215932203](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002215932203.png)

访问http://localhost:8081/dept/get/1和http://www.kuangstudy.com:9527/springcloud-provider-dept/dept/get/1都可以获得数据

![image-20201002220043786](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002220043786.png)

![image-20201002220035277](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002220035277.png)



在地址栏隐藏微服务springcloud-provider-dept的名称，application.yml中添加配置

```yaml
zuul:
  routes:
    mydept.serviceId: springcloud-provider-dept
    mydept.path: /mydept/**
```

访问这个地址即可http://www.kuangstudy.com:9527/mydept/dept/get/1

但是原路径http://www.kuangstudy.com:9527/springcloud-provider-dept/dept/get/1也能访问

![image-20201003104704709](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003104704709.png)

继续配置application.yml,原来的http://www.kuangstudy.com:9527/springcloud-provider-dept/dept/get/1不能访问了

```yaml
zuul:
  routes:
    mydept.serviceId: springcloud-provider-dept
    mydept.path: /mydept/**
  ignored-services: "*"  # 不能再使用某个(*：全部)路径访问了，ignored ： 忽略,隐藏全部的
```

![image-20201003105303720](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003105303720.png)

继续向application添加公共的访问前缀,访问路径变为http://www.kuangstudy.com:9527/kuang/mydept/dept/get/1

```yaml
zuul:
  routes:
    mydept.serviceId: springcloud-provider-dept
    mydept.path: /mydept/**
  ignored-services: "*"  # 不能再使用某个(*：全部)路径访问了，ignored ： 忽略,隐藏全部的~
  prefix: /kuang # 设置公共的前缀,实现隐藏原有路由
```

![image-20201003105852674](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003105852674.png)

## 10. Spring Cloud Config 分布式配置

### 概述

**分布式系统面临的–配置文件问题**

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务，由于每个服务都需要必要的配置信息才能运行，所以一套集中式的，动态的配置管理设施是必不可少的。spring cloud提供了configServer来解决这个问题，我们每一个微服务自己带着一个application.yml，那上百个的配置文件修改起来，令人头疼。

**什么是SpringCloud config分布式配置中心？**

![image-20201003110343457](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003110343457.png)

spring cloud config 为微服务架构中的微服务提供集中化的外部支持，配置服务器为各个不同微服务应用的所有环节提供了一个**中心化的外部配置**。

 spring cloud config 分为**服务端**和**客户端**两部分。

 服务端也称为 **分布式配置中心**，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密，解密信息等访问接口。

 客户端则是**通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息**。配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理。并且可用通过git客户端工具来方便的管理和访问配置内容。

**spring cloud config 分布式配置中心能干嘛？**

- 集中式管理配置文件
- 不同环境，不同配置，动态化的配置更新，分环境部署，比如 /dev /test /prod /beta /release
- 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
- 当配置发生变动时，服务不需要重启，即可感知到配置的变化，并应用新的配置
- 将配置信息以REST接口的形式暴露

**spring cloud config 分布式配置中心与GitHub整合**

### 入门案例

#### **服务端**

编写application.yml提交到github上或者码云上面（<font color=red>注意：---和空格的输入，否则之后访问不到</font>）

```yaml
spring:
  profiles:
    active: dev

---
spring:
  profiles: dev
  application:
    name: springcloud-config-dev

---
spring:
  profiles: test
  application:
    name: springcloud-config-test
```

![image-20201003142118117](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003142118117.png)

新建springcloud-config-server-3344模块导入pom.xml依赖

```xml
<dependencies>
    <!--web-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--config-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
        <version>2.2.5.RELEASE</version>
    </dependency>

    <!--eureka-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>
</dependencies>
```

resource下创建application.yml配置文件，Spring Cloud Config服务器从git存储库（必须提供）为远程客户端提供配置：

```yaml
server:
  port: 3344

spring:
  application:
    name: springcloud-config-server
  # 连接github远程仓库
  cloud:
    config:
      server:
        git:
          # 注意是https的而不是ssh
          uri: https://github.com/lzh66666/spring-cloud-kuang.git
          # 通过 config-server可以连接到git，访问其中的资源以及配置~
          default-label: main

# 不加这个配置会报Cannot execute request on any known server 这个错：连接Eureka服务端地址不对
# 或者直接注释掉eureka依赖 这里暂时用不到eureka
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

<font color=red>注意：default-label属性，默认是master提交，我改成main提交之后页面死活出不来</font>

![image-20201003141429024](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003141429024.png)

可以输入`git status`查看自己的分支

![image-20201003142733336](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003142733336.png)

访问：http://localhost:3344/application-dev.yml页面

![image-20201003141540570](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003141540570.png)

访问：http://localhost:3344/application-test.yml页面

![image-20201003142203950](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003142203950.png)

**HTTP服务具有以下格式的资源：**（main是我的分支，默认为master）

```java
/{application}/{profile}[/{label}]   		//	http://localhost:3344/application/test/main
/{application}-{profile}.yml				//	http://localhost:3344/application-test.yml
/{label}/{application}-{profile}.yml		//	http://localhost:3344/main/application-test.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

#### **客户端**

将本地git仓库springcloud-config文件夹下新建的config-client.yml提交到github或码云仓库：（千万别加注释，否则路径找不到）

```yaml
spring:
  profiles:
    active: dev
---
server:
  port: 8201
spring:
  profiles: dev
  application:
    name: springcloud-provider-dept

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
---
server:
  port: 8202
spring:
  profiles: test
  application:
    name: springcloud-provider-dept

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

![image-20201003143926010](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003143926010.png)

新建一个springcloud-config-client-3355模块，并导入依赖

```xml
<dependencies>
    <!--web-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!--config-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>

    <!--actuator-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

resources下创建application.yml和bootstrap.yml配置文件

- bootstrap.yml是系统级别的配置

  ```yaml
  # 系统级别的配置
  spring:
    cloud:
      config:
        name: config-client # 需要从git上读取的资源名称，不要后缀
        profile: dev
        label: main
        uri: http://localhost:3344
  
  ```

- application.yml是用户级别的配置

  ```yaml
  # 用户级别的配置
  spring:
    application:
      name: springcloud-config-client
  ```

创建controller包下的ConfigClientController.java用于测试

```java
@RestController
public class ConfigClientController {
    @Value("${spring.application.name}")
    private String applicationName; //获取微服务名称

    @Value("${eureka.client.service-url.defaultZone}")
    private String eurekaServer; //获取Eureka服务

    @Value("${server.port}")
    private String port; //获取服务端的端口号


    @RequestMapping("/config")
    public String getConfig(){
        return "applicationName:"+applicationName +
                "eurekaServer:"+eurekaServer +
                "port:"+port;
    }
}
```

主启动类

```java
@SpringBootApplication
public class ConfigClient_3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient_3355.class,args);
    }
}
```

测试：

启动服务端Config_server_3344 再启动客户端ConfigClient

访问：http://localhost:8201/config/

![image-20201003152213118](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003152213118.png)

### **小案例**

**本地新建config-dept.yml和config-eureka.yml并提交到码云仓库**

- config-dept.yml

  ```yaml
  spring:
    profiles: 
      active: dev
  ---
  server:
    port: 8081
  
  mybatis:
    type-aliases-package: nuc.ss.springcloud.pojo
    config-location: classpath:mybatis/mybatis-config.xml
    mapper-locations: classpath:mybatis/mapper/*.xml
  
  spring:
    profiles: dev
    application:
      name: springcloud-config-dept
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: org.gjt.mm.mysql.Driver
      url: jdbc:mysql://localhost:3306/db01?characterEncoding=utf-8&useUnicode=true
      username: root
      password: admin
  
  eureka:
    client:
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
    instance:
      instance-id: springcloud-provider-dept-8081
  
  info:
    app.name: lzh-springcloud
    company.name: com.lzh
  
  ---
  server:
    port: 8081
  
  mybatis:
    type-aliases-package: nuc.ss.springcloud.pojo
    config-location: classpath:mybatis/mybatis-config.xml
    mapper-locations: classpath:mybatis/mapper/*.xml
  
  spring:
    profiles: test
    application:
      name: springcloud-config-dept
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: org.gjt.mm.mysql.Driver
      url: jdbc:mysql://localhost:3306/db02?characterEncoding=utf-8&useUnicode=true
      username: root
      password: admin
  
  eureka:
    client:
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
    instance:
      instance-id: springcloud-provider-dept-8081
  
  info:
    app.name: lzh-springcloud
    company.name: com.lzh
  
  ```

- config-eureka.yml

  ```yaml
  spring:
    profiles: 
      active: dev
  ---
  server:
    port: 7001
  
  spring:
    profiles: dev
    application:
      name: springcloud-config-eureka
  
  eureka:
    instance:
      hostname: eureka7001.com
    client:
      register-with-eureka: false
      fetch-registry: false
      service-url:
        defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  ---
  server:
    port: 7001
  
  spring:
    profiles: test
    application:
      name: springcloud-config-eureka
  
  eureka:
    instance:
      hostname: eureka7001.com
    client:
      register-with-eureka: false
      fetch-registry: false
      service-url:
        defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  ```

- 上传成功
  ![image-20201003191627269](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003191627269.png)

**新建springcloud-config-eureka-7001模块，并将原来的springcloud-eureka-7001模块下的内容拷贝的该模块。**

- 清空该模块的application.yml配置

  ```yaml
  spring:
    application:
      name: springcloud-config-eureka-7001
  ```

- 并新建bootstrap.yml连接远程配置

  ```yaml
  spring:
    cloud:
      config:
        name: config-eureka # 仓库中的配置文件名称
        label: main
        profile: dev
        uri: http://localhost:3344
  ```

**在pom.xml中添加spring cloud config依赖**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>
```

**主启动类**

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServer_7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer_7001.class,args);
    }
}
```

**测试**

- 启动 Config_Server_3344，并访问 http://localhost:3344/master/config-eureka-dev.yml 测试
  ![image-20201003202046005](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003202046005.png)
- 启动ConfigEurekaServer_7001，访问 http://localhost:7001/ 测试
  ![](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003202114235.png)
- 显示上图则成功

新建springcloud-config-dept-8081模块并拷贝springcloud-provider-dept-8081的内容

同理导入spring cloud config依赖、清空application.yml 、新建bootstrap.yml配置文件并配置

```yaml
spring:
  cloud:
    config:
      name: config-dept
      label: main
      profile: dev
      uri: http://localhost:3344
```

主启动类

```java
//启动类
@SpringBootApplication
@EnableEurekaClient //在服务启动后自动注册到Eureka中
@EnableDiscoveryClient //服务发现~
@EnableCircuitBreaker//添加对熔断的支持
public class DeptProvider_8081 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider_8081.class,args);
    }

    //增加一个 Servlet
    @Bean
    public ServletRegistrationBean hystrixMetricsStreamServlet(){
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
        //访问该页面就是监控页面
        registrationBean.addUrlMappings("/actuator/hystrix.stream");
        return registrationBean;
    }
}
```

只需更改github远程即可实现部署

