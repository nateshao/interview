## **Spring中的bean的作用域有哪些？**

1. `singleton`(单例)：唯一bean实例，Spring中的bean默认都是单例的。
2. `prototype`(原型)：每次请求都会创建一个新的bean实例。
3. `request`：每一次HTTP请求都会产生一个新的bean，该bean仅在当前**`HTTP request`**内有效。
4. `session`：每一次HTTP请求都会产生一个新的bean，该bean仅在当前**`HTTP session`**内有效。
5. `global-session`：在一个全局HTTP Session中，容器会返回Bean的同一个实例，仅在使用portlet上下文有效。

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=MmMxOWI4NjY4Mjk5MjczM2NiMmM4OTRiMzIxMjVhZWVfOVM4d1I4NlJuOGRibVV2dmFFNDhOVHZTN0xxVWE4MkRfVG9rZW46Ym94Y25PaXhQTXlGMVphYWJVZEloUzZJdXdoXzE2ODgzNTg2NTI6MTY4ODM2MjI1Ml9WNA)

## Spring中bean的生命周期

**实例化：**第 1 步，实例化一个 Bean 对象（也就是我们常说的 new）

**属性赋值：**第 2 步，为 Bean 设置相关属性和依赖

**初始化：**初始化的阶段的步骤比较多，5、6步是真正的初始化，第 3、4 步为在初始化前执行，第 7 步在初始化后执行，初始化完成之后，Bean就可以被使用了

**销毁：**第 8~10步，第8步其实也可以算到销毁阶段，但不是真正意义上的销毁，而是先在使用前注册了销毁的相关调用接口，为了后面第9、10步真正销毁 Bean 时再执行相应的方法

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=Mjg0NmNmMjBhMTc4MDQ1MDA5ZTU2MTk3MWU2YjUxYTVfOTJkT0plNnY1bFlQaWRneVgxSTFabDBjcnpCbzFSdHlfVG9rZW46Ym94Y25meTFHbnhGSDhOZ1VoYVJEb2g5RzJlXzE2ODgzNTg2NTI6MTY4ODM2MjI1Ml9WNA)

## Spring怎么解决循环依赖问题?

循环依赖问题是指：类与类之间的依赖关系形成了闭环，就会导致循环依赖问题的产生。

- 例如：A类依赖了B类，B类依赖了C类，而最后C类又依赖了A类，这样就形成了循环依赖问题。

spring对循环依赖的处理有三种情况:

1. 单例模式下的setter循环依赖：通过”**三级缓存**”处理循环依赖。
   1. **singletonObjects 一级缓存**：用于保存实例化、注入、初始化完成的bean实例
   2. **earlySingletonObjects 二级缓存**：用于保存实例化完成的bean实例
   3. **singletonFactories 三级缓存**：用于保存bean创建工厂，以便于后面扩展有机会创建代理对象

## spring中bean的线程安全问题

**Bean的****线程安全****分析**

- 对于**`prototype`**作用域的Bean，每次都创建一个新对象，也就是线程之间不存在Bean共享，因此不会有线程安全问题。
- 对于**`singleton`**作用域的Bean，所有的线程都共享一个单例实例的Bean，因此是存在线程安全问题的。

线程安全问题都是由**全局变量及静态变量**引起的。

- 若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说， 这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步，否则就可能影响线程安全。

1. 在@Controller/@Service/@Repository等容器中，默认情况下，scope值是单例-singleton的，也是线程不安全的。
2. 不要在@Controller/@Service/@Repository等容器中定义静态变量，不论是单例(singleton)还是多实例(prototype)，她都是线程不安全的。
3. 一定要定义变量的话，用ThreadLocal来封装，这个是线程安全的。
4. **对有状态的 bean 要使用 prototype**。
5. **作用域对无状态的 bean 使用 singleton 作用域**。

  最后两条很重要喔！

## Spring框架中都用到了哪些设计模式

1. 单例模式：Bean默认为单例模式
2. 代理模式：Spring的AOP功能用到了JDK的动态代理
3. 工厂模式：BeanFactory就是简单的工厂模式的体现，用来创建对象的实例
4. 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新

# spring注解，spring事务，传播机制，隔离级别

## Spring事务实现方式有哪些?

spring框架提供了两种事务实现方式：**编程式事务、声明式事务**

- **编程式事务**：在代码中进行事务控制。优点：精度高。缺点：代码耦合度高
- **声明式事务**：通过`@Transactional`注解实现事务控制

**一般我们很少使用编程式事务，更多的是使用@Transactional注解实现**。当使用了@Transactional注解后事务的自动功能就会关闭，由spring帮助实现事务的控制。

Spring的事务管理是通过AOP代理实现的，对被代理对象的每个方法进行拦截，在方法执行前启动事务，在方法执行完成后根据是否有异常及异常的类型进行提交或回滚。

- Spring AOP动态代理机制：Spring在运行期间会为目标对象生成一个代理对象，并在代理对象中实现对目标对象的增强。

SpringAOP通过两种动态代理机制，实现对目标对象执行横向植入的。

1. **JDK 动态代理**： Spring AOP 默认的动态代理方式，若目标对象实现了若干接口，Spring 使用 JDK 的 java.lang.reflect.Proxy 类进行代理。
2. **CGLIB 动态代理**： 若目标对象没有实现任何接口，Spring 则使用 CGLIB 库生成目标对象的子类，以实现对目标对象的代理。

声明式事务（@Transactional)基本原理如下：

配置文件开启注解驱动，在相关的类和方法上通过注解@Transactional标识。

**原理：**

Spring 在启动的时候会去解析生成相关的bean，这时候会查看拥有相关注解的类和方法，并且为这些类和方法生成代理，并根据`@Transaction`的相关参数进行相关配置注入，这样就在代理中为我们把相关的事务处理掉了（开启正常提交事务，异常回滚事务）

- 真正的数据库层的事务提交和回滚是通过`bin log`或者`redo log`实现的。

## **@Transactional 事务失效场景**

spring事务的原理是AOP，进行了切面增强，失效的根本原因是这个AOP不起作用了！常见情况有如下几种

1. 数据库**不支持事务**
2. **没有被spring管理**
3. 方法不是public的: **@Transactional 只能用于public的方法上**，否则事务不会失效,如果要用在非public方法上，可以开启AspectJ代理模式。
4. 异常被吃掉，事务不会回滚(或者抛出的异常没有被定义，默认为RuntimeException)

## Spring事务的传播规则

 使用方法： @Transactional(propagation = Propagation.REQUIRES_NEW)

默认为REQUIRED，比较常用的就是REQUIRED、REQUIRES_NEW。

- **REQUIRED：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。** 
- **REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。** 
- SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。 
- MANDATORY：支持当前事务，如果当前没有事务，就抛出异常。
- NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 
- NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。 
- NESTED：支持当前事务，如果当前事务存在，则执行一个嵌套事务，如果当前没有事务，就新建一个事务。

## 什么是Spring

Spring是一个轻量级Java开发框架，是解决企业级应用程序开发的业务逻辑层和其他各层的耦合问题，Spring的两个核心特性就是**依赖注入（DI）和面向切面编程（AOP）**

## 什么是IOC

Spring通过IoC容器实现对象耦合关系的管理，并实现依赖反转（IoC），将对象之间的依赖关系交给IoC容器，实现解耦。

- 没有ioc之前。有ABCD，四个对象之间相互使用，当我使用一个对象就要管理3个对象。

## 什么是Spring的依赖注入DI

**依赖注入**：需要创建一个对象，无须在代码中new创建，而是依赖外部的注入。 

通常，依赖注入可以通过三种方式完成，即:

- **构造函数**注入
- **setter** 注入
- **接口**注入

## **Bean的装配方式**

Spring容器支持多种形式的Bean装配方式，如基于**XML**的装配、基于**注解**的装配方式、以及基于**Java配置类**的装配；

Bean的3种装配方式：

### **1、**基于XML形式的装配

```XML
<bean id="postservice" class="com.bbs.service.impl.PostserviceImpl">  
</bean>
```

在`<bean>`中使用**autowired属性**完成依赖的自动装配，不再使用`<property>`手动注入setter方法中的依赖，简化了配置，减少了xml中的代码量。

### **2、基于注解形式的装配**

Spring常用的注解：

- @Autowired 按类型自动装配（`byType`）。
- @Resource 是`byName`、`byType`方式的结合。
- @Qualifier 按名称自动装配，不能单用，需要和@Autowired配合使用（byName）。

### **3、基于Java配置类的装配**

单独写一个配置类来配置Bean。Spring的Java配置是通过使用@Bean和@Configuration来实现。

```Java
@Configuration //表示这个类是用来配置Bean的
class Config{
    @Value("张三") String name;  
    //配置一个Bean，相当于xml中的一个<bean>
    @Bean(name = "student") 
    public Student student(){
        //...具体业务逻辑
        return student;
    } 
}
```

## SpringMVC工作原理

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=OWE4NGYwOTRjYjcyMWRiMTM4YzhlYWY4YjFhYmRjODNfR0VOeGVVZHVaM2pDZENySEk0cFlObGtnVHVVVmpkaDVfVG9rZW46Ym94Y25zUkhCQWptS3M4Q0EzTUIwYVBYd05mXzE2ODgzNTg2NTI6MTY4ODM2MjI1Ml9WNA)

1. 客户端发送请求到 DispatcherServlet。
2. DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler(Controller层)。
3. HandlerAdapter 会处理 Handler的请求，处理后，就返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
4. ViewResolver 会根据逻辑 View 查找实际的 View。
5. DispaterServlet 把返回的 Model 传给 View（视图渲染）。View 返回给请求者（浏览器）。

## SpringMVC的主要组件 

**前端****控制器（DisatcherServlet）：**接收请求，响应结果，返回可以是json,String等数据类型，或者页面（Model）。 

**处理器映射器（HandlerMapping）：**根据URL去查找处理器，一般通过**xml****配置**或者**注解**进行查找。 

**处理器（****Handler****）**：就是我们常说的controller控制器啦，由程序员编写。 

**处理器适配器（HandlerAdapter）：**可以将处理器包装成适配器，这样就可以支持多种类型的处理器。 

**视图解析器（ViewResovler）：**进行视图解析，返回view对象（常见的有JSP,FreeMark等）。

## @RestController和@Controller的区别

1. @RestController = @Controller + @ResponseBody
2. @Controller是返回的jsp,html页面，@Restcontroller用的是jackson视图。
3. @Controller若返回JSON，XML或自定义mediaType内容到页面，则需要加@ResponseBody注解

## Spring Boot Starter的工作原理是什么?

Spring Boot在启动的时候会干这几件事情：

- Spring Boot启动会根据Starter包中寻找resources/META-INF/spring.factories文件，然后根据文件中配置的Jar包去扫描项目所依赖的Jar包。
- 根据spring.factories配置加载AutoConfigure类。
- 根据@Conditional注解的条件，进行自动配置并将Bean注入Spring Context。

总结一下，其实就是Spring Boot在启动的时候，读取Spring Boot Starter的配置信息，再根据配置信息对资源进行初始化，并注入到Spring容器中。