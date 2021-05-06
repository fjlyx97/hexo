---
title: Java基础
date: 2021-04-20 11:26:14
tags:
- 编程
- java
categories : "java家族"
---

> - Java入门基础
> - 从零开始学习Java

<!--more-->

# 基础
1. Java文件名必须和public class名称一致
2. Java文件可以有多个class，public class**只能有一个**
3. /** */ Java文档注释，可以使用JavaDoc生成文档
4. luyten java反编译工具
5. long类型最好在末尾增加L，好区分
6. float类型有精度，**只能在小数点后七位**，必要使用BigDecimal
7. final关键字代表常量获最终变量，不可修改。
8. 数组属于引用类型，创建完成数组后，默认会初始化。数组变量是在栈上，指向堆上的数据。
9. 成员变量有初始值，局部变量没有

# 引用类型
Java中，对象和数组，都是通过引用进行操作的

# 类
1. 默认有无参构造函数。
2. 如果定义了有参构造函数，无参构造函数就失效。
3. 可以定义多个有参构造函数
4. 当构造方法中，需要调用其他构造方法时，可以直接使用**this()**，但是必须位于第一行
5. 只能单继承
6. super调用父类构造方法，不写默认会自动添加。所有类的父类都是Object
7. Override重写 Overload重载
8. 静态方法可以被继承，但是不能被重写
9. ==比较基本数据类型是否相等，或者两个引用地址是否相等。equal比较对象内容是否相等。

## 内部类
内部类轻松访问外部类的私有属性
1. 如果内部类和外部类属性相同，可以使用out.this访问外部成员变量
2. Thread中的new Runnable就是匿名内部类

# 异常
异常出现时，try代码块往下不继续执行
- 尽量try包含的内容少一些
- 追踪堆栈可以从异常的最后开始追踪
## finally
无论是否触发异常，都会被执行。一般IO流关闭操作、数据库连接关闭操作设置在里面。

## 访问控制
Java有成员变量四种访问控制权限：public,protected,default,private
- public: 所有都可以访问
- protected:当前类、当前包、子类
- default: 当前类，当前包
- private： 当前类

Java类有两种访问权限：public,default
- public:同一个项目所有类
- default:被同一个包中所有类

## static关键字
- 类中定义的static变量，所有类成员共享
- 位于方法区的静态存储区
- 一般工具类方法定义成static

# 接口
Java中的继承关系是单继承，如果拥有多个父类，可以考虑使用接口。接口代表了一种能力。
1. 接口不能被实例化
2. 接口所有方法都是public abstract
3. 接口所有变量都是静态变量(static final)
4. 接口的所有方法，类中必须全部实现
5. 接口可以添加default方法，这样类就不用实现

# 代码块
静态代码块->构造代码块->普通代码块
## 构造代码块
```Java
class App() {
    {
        System.out.println("构造代码块");
    }
}
```
构造代码块会被添加到构造函数的开头，因此先执行。

## static代码块
数据库连接等其他需要提前准备好的代码，会放在static代码块中，只会执行一次
```Java
class App() {
    static {
        System.out.println("构造代码块");
    }
}
// 多次载入这个类，只会执行一次
```

## 同步代码块
多线程会使用，用来给共享空间加锁

# package
- 解决类重名问题
- 方便管理类
- 如果导入了两个同名的包，只能通过package+完整名称识别
- import static .... 静态导包，导入所有静态方法

## 常见的包
- java.lang Java语言核心类，提供String,Math,Integer,System,Thread等常用功能（不需要手动导入）
- java.awt 主要用于构造gui
- net 网络包
- util 工具包
- io 输入输出流

# Lambda表达式
Lambda必须和Interface结合在一起，可以采用()->{};方式进行创建
```
BiFunction<Integer,Integer,String> bi = (a,b)->"Hello"+a+b;
System.out.println(bi.apply(10, 23));
```

## 接口申明
- Supplier 一个输出
- Consumer 一个输入
- Function 一个输入一个输出（输入输出类型不同）
- UnaryOperator 一个输入一个输出（输入输出类型相同）

# Stream Api
stream是一组用来处理数组、集合的API
- 告别for循环
- 多核友好

1. 不是数据结构，没有内部存储
2. 不支持索引访问
3. 延迟计算
4. 支持并行
5. 易生成数组
6. 支持过滤、查找、转换汇总

# 注解(Annontation)
- 类似修饰符，应用于包、类型、构造方法、方法、成员变量、参数及本地变量中。
- Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析使用
- 不会影响到代码逻辑，仅仅起到辅助性作用

## 元注解
### 自定义注解
@interface 注解名{}

### 元注解
负责注解其他注解，有四个元注解
- @target(重要):限制注解的添加位置
```
@Target(ElementType.Method,ElementType.Type)
```
- @retention(重要):限制注解识别时间，编译、运行等(source->class->runtime)
- @document:说明javadoc是否要包含该注解
- @inherited：表示该注解能否被继承

### 参数
定义的方式看起来像方法，实际是填写参数名称
```
@interface MyAnnotation {
    String name() default "123";
    int age();
    // 默认方法名称是value
    // 可以添加默认值
}

@MyAnnotation(name="123",age=12)
```

# Collection
- add 添加元素，必须是Object对象，普通类型会被自动拆包装包
- addAll 添加另一个集合元素
- clear 只清空元素，但是对象并没有被回收


# ArrayList 和 Vector
- Vector线程安全，但是效率低
- ArrayLit扩容1.5倍，Vector为原来的2倍

# Spring
Spring是一个**IOC**和**AOP**的容器框架

## IOC控制反转
1. 谁控制谁：早期由我们自己去New。有了IOC后，由Ioc容器给我们对象
2. 什么叫反转：由主动创建，变成了被动接受，这就是反转

- IOC和DI是从不同角度描述问题，IOC是设计思想，DI是具体的实现方式
- Spring把IOC设计理念应用到了框架中，而不是它提出的

## Bean
### 基础
- Bean创建的对象默认都是单例的，可以通过scope来指定
- 获取对象主要是通过id和type
```xml
<bean id="person" class="com.codeforme.bean.Person">
        <property name="age" value="10"></property>
        <property name="height" value="20"></property>
        <property name="weight" value="30"></property>
</bean>
```
- 当需要从容器中获取对象时，最好保留无参构造方法，方便调用。可以通过<constructor-arg>来构造变量。正常使用时，都是通过name直接指定即可
- 可以通过parent属性，继承父类的属性值
- bean对象被创建的时候，是通过xml配置文件定义的顺序创建的。可以使用depends-on属性

### p命名空间
通过xmlns:p(xml name space)引入p命名空间，引入后可以通过**p:属性名**，来为属性赋值（用的较少）

### 复杂数据赋值
这部分可以查阅官方文档

### Bean对象的作用域
- 通过scope指定
prototype（多例）,singleton(单例),request(每次发送请求都会有一个新对象,几乎不用),session(每一次会话都会有一个新对象,几乎不用)
- 新版本只剩prototye,singleton

### 利用工厂模式创建Bean对象
```java
// 静态工厂
<bean id="person" class="静态工厂的名" factory-method="方法名">
    // 如果有构造方法的话
    <constructor-arg>
</bean>

// 实例工厂
// 先创建工厂实例Bean，然后在调用工厂实例方法
// factory-bean指定工厂实例
// factory-method指定工厂方法
```

### 继承FactoryBean创建对象（一种补充，不常用）
1. implements FactoryBean<类名>
2. 把对象交给容器，用bean的方式
- 用到的时候才会创建，不会在启动的时候创建

### 创建对象指定调用的方法
- init-method，在对象创建完成之后会调用初始化方法
- destory-method，在容器关闭的时候，调用销毁方法
- prototype的话，初始化方法会调动，销毁方法不会调用

### 配置Bean对象的初始化方法(init前后)
implements BeanPostProcessor

### 创建第三方bean对象
1. maven导入相关的包
2. class指定jar包要创建的类
- 数据库配置，可以引入配置文件。通过context命名空间（需要注意**自带的系统变量**）

### 基于xml文件自动装配
基于autowire属性
- default/no: 不装配
- byName: 按ID装配，根据set方法决定
- byType: 按类型装配，以类型去配置文件中查找对象
- constructor(基本不用):先按类型判断、多个类型按名字

### SPEL表达式语言
查看官方文档

### 注解使用
常用的四个注解写在类上面，都可以完成注册Bean的功能
- @Component:组件，理论上可以在任意的类上进行添加，在扫描的时候都会完成Bean的注册
- @Controller: 放置在控制层，用来接受用户的请求
- @Service: 放置在业务逻辑层
- @Repository: 放置在数据访问层
- @Bean: 自定义Bean

这四个注解运行过程中，spring不会对他们进行区分（开发过程中保证代码可读性）。

#### 基础使用
注解和xml相比，xml优先级更高
1. 配置文件配置
```xml
xmlns:context="导入命名空间"
<context:component-scan base-package="包名">
    // 更细粒度的控制，可以选择扫描哪个注解，或者不扫描哪个注解
    <context:include-filter type="" expression="">
    // type为类型
    // assignable 类的完整限定名
    // annotation 排除注解
    // regex 正则表达式（一般不用）
    // aspectj aop方式（一般不用）
    // custom 自定义方式，可以自己定义自己的筛选规则（一般不用）
</context:component-scan>
```
2. 对应的类名添加@Component
3. getBean可以获取（名称默认是驼峰，开头小写）

- 可以通过@Scope配置是否为单例hhh:w

#### Autowired
默认情况下，按照类型ByType来进行装配，如果找到两个，使用ByName来装配
- 使用@Qualifier("名称")，可以指定名称
- 使用@Resource可以完成和@Autowired一样的功能。@Resource是jdk提供的功能，因此可以在其他框架中使用
- 默认@Resource通过名称来装配，@Autowired通过类型来装配

#### 依赖范型注入
具体看资料

## AOP面向切面编程
程序运行期间，将某段代码动态切入到指定方法，指定位置进行运行

1. 进行注解切面，@Aspect，@Component
2. 配置文件开启注解，**需要扫描注解**

- @Before:前置通知，在方法执行前完成
- @After:后置通知，方法执行后通知
- @AfterReturning 在返回结果后通知
- @AfterThrowing:异常通知
- @Around 环绕通知

```java
public class LogUtil {
    @Before("execution(public String com.codeforme.bean.Person.toString())")
    public void begin() {
        System.out.println("Log begin");
    }

    @After("execution(public String com.codeforme.bean.Person.toString())")
    public void end() {
        System.out.println("Log end");
    }
}

// 或者在配置文件中使用
<aop:config>
    <aop:aspect ref="bean名称">
    </aop:aspect>
</aop:config>
```

### *通配符
- 采用的是最精确的匹配方式，但是不常用
- 实际使用更多的是通配符*
- *在匹配时，只能匹配一层路径
- *不能匹配访问控制符（public,private..)，如果不确定，可以直接不写

### ..匹配符
- 可以匹配多个参数，任意类型
- 可以匹配多层路径

### 逻辑运算
包含&&,||,!

### 执行顺序
- 正常情况:@Before->@After->@AfterReturning
- 异常情况:@Before->@After->@AfterThrowing

### 参数
如果用了Aop，参数就不能乱写，统一用JoinPoint

AfterReturning可以通过注解Returing来获取到返回值

### 重复的execution表达式
可以通过@Pointcut("表达式")，配上public void方法，即可创建通用表达式

### 环绕通知(推荐使用)
环绕通知的参数需要传入ProceedingJoinPoint，其中最关键的方法是proceed方法。并且返回值要为Object
调用proceed方法时，相当于执行原函数的方法（method.invoke），同时可以自己修改结果值，较为方便。
- 环绕前置->@Before->环绕后置->环绕返回->@After->@AfterReturning

### 切面顺序
可以通过Order指定权重，越小越先。默认是通过首字母来判断

### 数据库事务
#### 声明式事务
使用@Transaction注解，但是其中参数配置有多种。
- timeout超时时间，单位是秒
- readonly只读事务
- noRollbackFor，指定某些异常不用回滚
- rollbackFor，需要指定，才会回滚
- rollbackForClassName
- isolation隔离级别
- propagation传播特性

1. **如果异常被捕获，不会回滚(最常见）**
2. 方法不是public的，不会被监控到
3. 同一个类的方法调用，不会回滚

#### 传播特性propagation
一个事务方法被另一个事务方法调用（一个事务套一个）

##### Required（常用）
如果有事务运行，调用的方法也会在这个事务里运行。

##### Required_new（常用）
运行的时候，不管外面是否有事务，都会启动新事务

##### Supports
如果事务在运行，当前的方法就在这个事务运行，否则可以不运行在事务中。

##### Not_support
不允许运行在事务中，如果外层有事务，将它挂起

##### Never
当前方法不允许运行在事务中，否则抛出异常

##### Mandatory
必须运行在事务中，否则抛出异常

##### NESTED
对比Required，会多一个嵌套事务

# Spring MVC
1. 类上加@Controller
2. 方法加@RequestMapping
```
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public String handleRequest(Map<String,String> map) {
        System.out.println("YEsssss");
        map.put("hello","Yesss");
        // return jsp name
        return "hello";
    }
}
```

# Spring boot
## 自定义配置文件
可以在yml或者property里面定义属性，通过：
```
@Value(${完整属性名来导入})
```

配置文件中也可以通过以下方案互相引用：
```
age=10
height=${age}
```

配置文件可以使用random作为随机值

### 多环境配置
文件名需要满足application-{profile}.properties，在主配置文件application.properties进行加载。
```
spring.profiles.active=dev
```

## RESTfuf API
- @Controller 修饰class，用来处理http请求
- @RestController 默认返回json格式
- #RequestMapping 配置Url映射
```
@RequestMapping(value="/hello",method=RequestMethod.GET)
```

# Mybatis
## settings
```
https://mybatis.org/mybatis-3/zh/configuration.html
```
可以查看链接获取所有设置

## XML mapper 属性
### insert
- id：唯一标识符，和代码的接口对应
- parameterType：限定参数类型
- flushCache:标示当前sql语句的结果是否接入二级缓存
- statementType（相对重要）：用来选择执行sql语句的方式，可以用statement（基本jdbc操作，用来表示sql语句，不可以防止sql注入）,prepared(预编译，防止sql注入),callable(调用存储过程)
- useGeneratedKey:是否使用自增主键
```
useGeneratedKeys=”true” keyProperty=”id” 
```

### select
- 当返回结果是一个集合的时候，直接使用resultType
- 关联查询时，需要使用resultMap
- #{}使用预编译（不会出现sql注入，推荐使用）、${}使用正常的拼接
- 如果查询单个参数的时候，#{para}，里面的参数随便写
- 可以通过resultMap属性，进行字段映射，比如数据库为dname映射为代码中的name

### 缓存
- 一级缓存，session级别缓存
- 二级缓存，全局缓存，所有session共享
- 第三方缓存

某些情况下，一级缓存可能会失效。例如开启了多个会话、传递对象时属性变更了、当在任意一个会话中改变了数据、手动clear了缓存

二级缓存表示全局缓存，必须等session关闭了才会生效