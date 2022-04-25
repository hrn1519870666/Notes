### Spring

**Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器（框架）。**



#### IoC（控制反转）

将原本在程序中手动创建对象的控制权，交由Spring的IoC容器来管理。**通俗理解：对象由Spring 来创建 , 管理 , 装配。 IoC 容器就像是一个工厂，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，不用考虑对象是如何被创建出来的。** 

```xml
使用Spring来创建对象，在Spring中，这些对象称为Bean

类型 对象名 = new 类型()
User user = new User()

id = 对象名
class = 对象所属的类
property 给对象中的属性赋值

<bean id="hello" class="com.kuang.pojo.Hello">
    <property name="name" value="Spring"/>
</bean>
```



#### DI（依赖注入）

- 依赖 : 指Bean对象的创建依赖于容器。
- 注入 : 指Bean对象中的所有属性，由容器来注入。

1.常量注入

```xml
 <bean id="student" class="com.kuang.pojo.Student">
     <property name="name" value="小明"/>
 </bean>
```

2.Bean注入

这里的值是一个引用：ref

```xml
 <bean id="addr" class="com.kuang.pojo.Address">
     <property name="address" value="重庆"/>
 </bean>
 
 <bean id="student" class="com.kuang.pojo.Student">
     <property name="name" value="小明"/>
     <property name="address" ref="addr"/>
 </bean>
```

3.数组、List、Map等注入方式，见笔记03。



**使用注解注入属性**

在属性上添加@value("值")

```java
@Component("user")
// 相当于配置文件中 <bean id="user" class="当前注解的类"/>
public class User {
   @Value("秦疆")
   // 相当于配置文件中 <property name="name" value="秦疆"/>
   public String name;
}
```



#### AOP（面向切面编程）

**AOP基于动态代理。**能够将那些与业务无关，**却为业务模块所共同调用的逻辑（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**。



#### **Bean的作用域**

bean就是由IoC容器初始化、装配及管理的对象。

| 类别      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| singleton | 在Spring IoC容器中仅存在一个Bean实例，Bean以单例方式存在，默认值 |
| prototype | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行new XxxBean() |



**配置Bean的方式：XML，注解，Config配置类。**

#### 将一个类声明为Spring的 bean 的注解有哪些?

**@Component和它的三个衍生注解：**

- @Controller：controller层
- @Service：service层（在实现类ServiceImpl上加注解）
- @Repository：dao层

写上这些注解，就相当于将这个类交给Spring管理装配。



#### **基于Config配置类进行配置**

1.实体类：

```java
@Component  //将这个类标注为Spring的一个组件，放到容器中
public class Dog {
   public String name = "dog";
}
```

2.新建一个config配置包，编写一个MyConfig配置类：

```java
@Configuration  //代表这是一个配置类，和beans.xml作用一样
public class MyConfig {
    
   // 注册一个Bean，相当于bean标签
   @Bean
   // 方法名dog相当于bean标签中的id属性
   // 方法返回值类型Dog相当于bean标签中的class属性
   public Dog dog(){
       return new Dog();
  }

}
```



#### **@Autowired**

默认使用的是byType的方式注入相应的Bean。例如：

```java
@Autowired
private UserService userService;
```

这段代码会在初始化的时候，在spring容器中寻找一个类型为UserService的bean实体注入，关联到userService的引入上。

可以结合@Qualifier注解实现byName的方式：

```java
@Autowired
@Qualifier(value = "userService2")
private UserService userService3;
```




#### @Component 和 @Bean 的区别？

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。
3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

`@Bean`注解使用示例：

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

 上面的代码相当于下面的 xml 配置

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```



#### Spring 中的 bean 生命周期?

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个Bean的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含  init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。



#### Spring 框架中用到了哪些设计模式？

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。
- ......



#### Spring 事务中哪几种事务传播行为?

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。



**补充：import标签**

可以将多个配置文件导入合并为一个，使用时用总的配置。

```xml
<-- applicationContext.xml -->

<import resource="bean1.xml"/>
<import resource="bean2.xml"/>
<import resource="bean2.xml"/>
```



**笔记重点：**

01：IOC理论推导~结束

03：Bean的作用域（在最后）

06：静态/动态代理（理解思想即可，代码实现不用深究）

07：什么是AOP，第一种和第三种使用方式



**视频重点：**

3，4，5，19（动态代理）







### Spring MVC

#### 说说自己对于 Spring MVC 的了解?

MVC 是一种设计模式，Spring MVC 是一种 MVC 框架。它可以简化Web层的开发。在Spring MVC 下我们一般把后端项目分为 Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层(控制层，返回数据给前台页面)。

**简单原理图：**

![](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/60679444.jpg)



#### SpringMVC 工作原理

![image-20220403114619393](C:\Users\黄睿楠\AppData\Roaming\Typora\typora-user-images\image-20220403114619393.png)

**流程说明（重要）：**

1. 客户端（浏览器）发送请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler `来调用真正的处理器来处理请求和相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）



**笔记重点：**

01：2.3、SpringMVC执行原理

04：ModelAndView，数据处理

08：拦截器







### SpringBoot

**Idea新建一个springboot项目之后可以删除的文件：**总共5个，mvn，git，HELP

![image-20220403103744962](C:\Users\黄睿楠\AppData\Roaming\Typora\typora-user-images\image-20220403103744962.png)



**自动装配：**

springboot所有自动配置都是在启动时扫描并加载：所有的自动类都在`spring.factories`里面，但不一定生效，需要判断条件是否成立，只要导入了对应的starter，就有对应的启动器了，自动装配就会生效。

**结论：**

1. SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值
2. 将这些值作为自动配置类导入容器 ， 自动配置类就生效 ， 帮我们进行自动配置工作；
3. 整个J2EE的整体解决方案和自动配置都在springboot-autoconfigure的jar包中；
4. 它会给容器中导入非常多的自动配置类 （xxxAutoConfiguration）, 就是给容器中导入这个场景需要的所有组件 ， 并配置好这些组件 ；
5. 有了自动配置类 ， 免去了我们手动编写配置注入功能组件等的工作。



**笔记重点：**

02，05（自动装配原理）

03：yaml注入配置文件

18：SpringSecurity
