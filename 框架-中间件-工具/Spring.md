## Spring

**Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器（框架）。**



### IoC（控制反转）

**将原本在程序中手动创建对象的控制权，交由Spring的IoC容器来管理。通俗理解：对象由Spring来创建 , 管理 , 装配。 IoC 容器就像是一个工厂，当我们需要创建一个对象的时候，只需要配置好配置文件或注解即可，不用考虑对象是如何被创建出来的。** 



[IoC理论推导和本质](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247484092&idx=1&sn=ab5bfb967cdd0b4268517e0339b12d61&scene=19#wechat_redirect)



### DI（依赖注入）

- 依赖 : 指Bean对象的创建依赖于容器。
- 注入 : 指Bean对象中的所有属性，由容器来注入。

#### 配置Bean的方式

XML，注解，Config配置类。

#### 使用XML注入属性

1.常量注入

```xml
// 使用Spring来创建对象，在Spring中，这些对象称为Bean

类型 对象名 = new 类型()
User user = new User()

id = 对象名
class = 对象所属的类
property 给对象中的属性赋值

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

3.数组、List、Map等注入方式，见笔记Spring03。

#### 使用注解注入属性

在属性上添加@value("值")

```java
// 相当于配置文件中 <bean id="user" class="当前注解的类"/>
@Component("user")
public class User {
    // 相当于配置文件中 <property name="name" value="秦疆"/>
   @Value("秦疆")
   public String name;
}
```

#### 哪些注解可以将一个类声明为Spring的 bean?

**@Component和它的三个衍生注解：**

- @Controller：controller层
- @Service：service层（在实现类ServiceImpl上加注解）
- @Repository：dao层

还有@Configuration ：声明该类为一个配置类，可以在此类中声明一个或多个 `@Bean` 方法。

写上这些注解，就相当于将这个类交给Spring管理装配。

#### 基于Config配置类进行配置

新建一个config配置包，编写一个MyConfig配置类：

```java
@Configuration  //代表这是一个配置类，和beans.xml作用一样
public class MyConfig {
    
   // 注册一个Bean，相当于bean标签
   @Bean
   // 方法名dog相当于bean标签中的id属性
   // 返回值类型Dog相当于bean标签中的class属性
   public Dog dog(){
       return new Dog();
  }

}
```

 上面的代码相当于下面的 xml 配置：

```xml
<beans>
    <bean id="dog" class="com.kuang.Dog"/>
</beans>
```


#### @Component（@repository...） 和 @Bean 的区别？

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。
2. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现（总不能在第三方类上加`@Component`注解）。



#### @Autowired

**默认使用的是byType的方式注入相应的Bean。**例如：

```java
@Autowired
private UserService userService;
```

这段代码会在初始化的时候，在spring容器中寻找一个类型为UserService的bean实体注入，关联到userService的引用上。

可以结合@Qualifier注解实现byName的方式：

```java
@Autowired
@Qualifier(value = "userService2")
private UserService userService2;
```



### Bean的作用域

bean就是由IoC容器初始化、装配及管理的对象。

|   类别    |                             说明                             |
| :-------: | :----------------------------------------------------------: |
| singleton | 在Spring IoC容器中仅存在一个Bean实例，Bean以单例方式存在，默认值 |
| prototype | **每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行new XxxBean()** |

#### Singleton

当一个bean的作用域为Singleton，那么Spring IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回同一个bean实例。**Singleton是单例类型，就是在创建容器时同时自动创建了一个bean的对象，不管你是否使用，他都存在，每次获取到的对象都是同一个对象。**

测试：

```java
 @Test
 public void test(){
     ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
     User user = (User) context.getBean("user");
     User user2 = (User) context.getBean("user");
     System.out.println(user==user2);
 }
```

#### Prototype

当一个bean的作用域为Prototype，表示一个bean定义对应多个对象实例。Prototype作用域的bean会在每次对该bean请求时都会创建一个新的bean实例。Prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象。根据经验，**对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。**在XML中将bean定义成prototype，可以这样配置：

```xml
 <bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>  
  或者
 <bean id="account" class="com.foo.DefaultAccount" singleton="false"/>
```



### Spring 中的 bean 生命周期?

Spring启动，查找并加载需要被Spring管理的bean，（通过反射）进行Bean的实例化。

如果涉及到一些属性值，利用 `set()`方法设置属性值。



如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。

如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例（与前面类似，如果实现了其他 `*.Aware`接口，就调用相应的方法）。



如果 Bean 实现了`BeanPostProcessor`接口，Spring就将调用他们的`postProcessBeforeInitialization()`方法。

如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。

如果 Bean 在配置文件中的定义包含  init-method 属性，执行指定的方法。

如果 Bean 实现了`BeanPostProcessor`接口，Spring就将调用他们的`postProcessAfterInitialization()`方法。



此时，Bean已经准备就绪，可以被应用程序使用。他们将一直驻留在应用上下文中，直到应用上下文被销毁。

当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

[Spring bean生命周期：类比人的一生](https://juejin.cn/post/7075168883744718856)

[Spring bean生命周期](https://juejin.cn/post/6844904065457979405)



### AOP（面向切面编程）

**AOP基于动态代理。**能够将那些与业务无关，**却为业务模块所共同调用的逻辑（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码，降低模块间的耦合度。**

[静态代理和动态代理](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247484130&idx=1&sn=73741a404f7736c02bcdf69f565fe094&scene=19#wechat_redirect)

[AOP的实现方式](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247484138&idx=1&sn=9fb187c7a2f53cc465b50d18e6518fe9&scene=19#wechat_redirect)



### Spring 框架中用到了哪些设计模式？

- **单例模式** : Spring 中的 Bean 默认都是单例的。
- **工厂模式** : Spring通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理模式** : Spring AOP 功能的实现。



- **装饰器模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。
- ......



### Spring 事务中哪几种事务传播行为?

七种：

- **PROPAGATION_REQUIRED** —— 支持当前事务，如果当前没有事务，则**新建一个事务，这是最常见的选择，也是 Spring 默认的一个事务传播属性。**
- **PROPAGATION_SUPPORTS** —— 支持当前事务，如果当前没有事务，则**以非事务方式执行。**
- **PROPAGATION_MANDATORY** —— 支持当前事务，如果当前没有事务，则**抛出异常。**
- **PROPAGATION_REQUIRES_NEW** —— 新建事务，如果当前存在事务，把当前事务挂起。
- **PROPAGATION_NOT_SUPPORTED** —— 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- **PROPAGATION_NEVER** —— 以非事务方式执行，如果当前存在事务，则抛出异常。
- **PROPAGATION_NESTED**--如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作，新建。

[声明式事务](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247484148&idx=1&sn=9d3edabf2443cd3a552e62e51b1f4097&scene=19#wechat_redirect)



**视频重点：**

3，4，5，17，19（动态代理），27





## Spring MVC

### 什么是Spring MVC

**MVC 是一种架构模式，Spring MVC 是一种 MVC 框架。**它可以简化Web层的开发。在Spring MVC 下我们一般把后端项目分为 Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层(控制层，返回数据给前端页面)。

**简单原理图：**

![](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/60679444.jpg)



### SpringMVC 工作原理

![img](https://img-blog.csdnimg.cn/img_convert/de6d2b213f112297298f3e223bf08f28.png)

流程说明：

1. 客户端（浏览器）**发送请求，** `DispatcherServlet`**拦截请求。**
2. `DispatcherServlet` 根据请求信息**调用** `HandlerMapping` 。`HandlerMapping` 根据 uri 去匹配**查找**能处理的 `Handler`（也就是我们平常说的 `Controller` 控制器）。
3. `DispatcherServlet` **调用** `HandlerAdapter`适配**执行** `Handler` 。
4. `Handler` 完成对用户请求的处理后，会返回一个 `ModelAndView` 对象给`DispatcherServlet`，`ModelAndView` 顾名思义，包含了数据模型以及相应的视图的信息。`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
5. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
6. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
7. 把 `View` 返回给请求者（浏览器）



[2.3、SpringMVC执行原理](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247483970&idx=1&sn=352e571ee88957ce391e972344e2a3d7&scene=19#wechat_redirect)



[结果跳转方式和数据处理](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247483998&idx=1&sn=97c417a2c1484d694c761a2ad27f217d&scene=19#wechat_redirect)



[拦截器（AOP思想的应用）](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247484026&idx=1&sn=eba24b51963e8c3293d023cbcf3318dc&scene=19#wechat_redirect)



**视频重点：**

5（SpringMVC执行原理）





## SpringBoot

### 自动装配

**springboot所有自动配置都在启动时扫描并加载：所有的自动类都在`spring.factories`里面，但不一定生效，需要判断条件是否成立，只要导入了对应的starter，就有了对应的启动器，自动装配就会生效。**

**结论：**

1. SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值
2. 将这些值作为自动配置类导入容器 ， 自动配置类就生效 ， 帮我们进行自动配置工作；
3. 整个J2EE的整体解决方案和自动配置都在springboot-autoconfigure的jar包中；
4. 它会给容器中导入非常多的自动配置类 （xxxAutoConfiguration）, 就是给容器中导入这个场景需要的所有组件，并配置好这些组件 ；
5. 有了自动配置类，免去了我们手动编写配置注入功能组件等的工作。



**Idea新建一个springboot项目之后可以删除的文件：**总共5个，3个mvn，git，HELP



[运行原理初探](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247483743&idx=1&sn=431a5acfb0e5d6898d59c6a4cb6389e7&scene=19#wechat_redirect)



[自动配置原理](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247483766&idx=1&sn=27739c5103547320c505d28bec0a9517&scene=19#wechat_redirect)



[yaml注入配置文件](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247483744&idx=1&sn=b4ec762e71b2ddf9403c035635299206&scene=19#wechat_redirect)



[SpringSecurity](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247483957&idx=1&sn=fc30511490b160cd1519e7a7ee3d4ed0&scene=19#wechat_redirect)
