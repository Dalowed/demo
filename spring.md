# Spring注解式开发  

注解式开发是为了简化XML的配置。Spring倡导全注解开发。  

#### 声明bean的注解开发  

```xml
<!--
        注解开发，要在配置文件中指定要扫描哪些包的类
        1.首先要添加配置
            xmlns:context="http://www.springframework.org/schema/context"
            http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd"
        2.contetxt标签  以下扫描bean包
-->

<context:component-scan base-package="com.spring6.bean"/>
```

- @Component
- @Controller      （表示层）
- @Service           （业务层）
- @Repository    （持久层）

==其实4个注解都是1个意思，另外3个是为了增强可读性。== 

如果注解没有写value值，默认是类名首字母小写。  

#### 解决多个包扫描问题

1. 写两个包，用逗号隔开
2. 如果有同一父包，可直接扫描父包，不过会牺牲一定效率

#### 选择性实例化Bean

#####  ==use-defaultfilter== 属性 

```xml
<!--
         use-default-filters=”false“
         如果为false表示所有注解不可用
         可以用标签context:include-filter来include(添加)

         如果为true表示所有注解可用
         可以用标签context:exclude-filter来exclude(排除)
    -->
    <context:component-scan base-package="com.spring6.bean" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
<!--        <context:exclude-filter type="annotation" expression=""/>-->
    </context:component-scan>
```



#### 负责注入的注解  

##### @Value注解

当类型是==简单类型== 时，可以用@Value注解注入

可以==不用写set方法和构造方法== 。如果有，也可以写在set方法或构造方法上  

##### @Autowired与@Qualifier  

@Autowired注解可以用来==注入非简单类型== 。被翻译为：自动连线的，或者自动装配的  

单独使用@Autowired注解，==默认根据类型装配== 。 （默认是byType）  

两个联合使用才进行进行名字装配。（ByName）  

如果单独使用@Autowired并且只有一个构造方法，可省略注解。（为提高程序可读性，不推荐使用）

###### 接口只有单个实现类时  

```java
//该类型不是简单类型，所以不能用@value注解注入
@Autowired
private OrderDao OrderDao;
```

###### 接口有多个实现类时  

```java
//当接口有多个实现类，spring就不知道引用哪个对象
//这时可使用@Qualifier指定引用哪个对象
@Autowired
@Qualifier("orderDaoImpForMysql")
private OrderDao OrderDao;
```

##### @Resource  

@Resource注解也可以完成非简单类型注入。那它和@Autowired注解区别  

- @Resource注解是JDK扩展包中，属于JDK一部分，更具有通用性。@Autowired注解是spring框架自己的。

- ==@Resource注解默认根据名称装配byName，未指定时，使用属性名作为name。通过name找不到再启动byType装配==  

- ==@Autowired注解默认根据类型装配byType，如果想根据名称装配，必须跟@Qualifier联合使用== 

- @Resource注解用在属性上，setter方法上  

- @Autowired注解用在属性上，setter方法上，构造方法上，参数列表前  

```xml
<!--
	高于jdk11或低于jdk8需要引入注解依赖
	spring6+为jakarta
	spring5-javax
-->
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
```

#### Spring全注解开发  

所谓的全注解开发就是不再用XML配置文件。写一个配置类代替配置文件来扫描  

```java
@Configuration
@ComponentScan("com.spring6.bean")
public class SpringConfig {
}
```

测试程序，用类代替配置文件。

```java
@Test
public void testNoXml(){
    //new ClassPathXmlApplicationContext("spring.xml");
    //因为没有xml文件，所以不能写上面的了。
    AnnotationConfigApplicationContext context 
            = new AnnotationConfigApplicationContext(SpringConfig.class);
    User user = context.getBean("user", User.class);  
```



# GoF之代理模式  

#### 代理模式的作用

- 当对象需要收到保护时，可以考虑使用代理对象去完成某个行为。  

- 当某个对象的功能进行功能增强时，可以考虑找一个代理进行增强。  
- ==A对象无法和B对象直接进行交互时，也可以使用代理模式解决==  

#### 代理模式在代码实现上分为：

- 静态代理  
- 动态代理  

##### 静态代理

```java
//方法扩展又符合OCP原则。
public void service() {
    System.out.println("提供服务");
}
```

  

```java
/*
	如果要扩展功能可以编写一个代理类，把实现类作为代理类的属性。
*/
private OrderService orderService = new OrderServiceImp();
public void service() {
    //这样不仅可以执行以前代码，也可以自己新增代码。
    System.out.println("模拟代码1");
    orderService.service();
    System.out.println("模拟代码2");
}
```

静态代理解决

优点：一定程度上降低了耦合度，也符合OCP原则。  

缺点：接口多久会导致类爆炸，代码得不到复用等等。

##### 动态代理  

举例：一个接口和一个实现类  

以下为

```java
//创建目标对象
        OrderService target = new OrderServiceImp();
//创建代理对象
        /*
        newProxyInstance创建代理对象
            本质上，这个Proxy.newProxyInstance的执行做了两件事
                1.在内存中动态生成了代理类的字节码class文件
                2.new对象，通过内存中生成的代理类字节码，实例化代理类

            关于三个参数
                ClassLoader loader
                    类加载器
                    jdk要求。目标类的类加载器必须和代理类的类加载器使用同一个
                Class<?>[] interfaces   代理类要实现的接口
                    代理类和目标类要实现相同接口
                    在生成代理类时需要告诉他实现哪些接口
                InvocationHandler h 调用处理器
                    InvocationHandler是一个接口
                    在调用处理器接口中编写增强代码
         */
        OrderService proxyObj =(OrderService)
                Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), new InvocationHandler(target));
        //调用目标方法
        proxyObj.service();
        proxyObj.exit();
```

  

再单独写一个类实现InvocationHandler接口，在invoke方法中编写增强代码  

```java
//通过构造器获取目标对象
    private Object target;
    public InvocationHandler(Object target) {
        this.target = target;
    }

    /*
            当代理对象调用代理方法时，注册在InvocationHandler接口中的invoke方法会被jvm自动调用
            我们的增强程序写在invoke里

            invoke方法三个参数
            Object proxy
                代理对象的引用
            Method method
                目标对象上的目标方法
            Object[] args
                目标方法上的实参
         */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("增强代码1");
        //调用目标方法
        Object retValue = method.invoke(target, args);
        System.out.println("增强代码2");
        //invoke的返回值，如果在对象调用代理方法后，需要返回值，invoke方法必须将目标方法执行结果进行返回
        //否则目标方法接收不到返回值。
        return retValue;
//        return null;
```

  

# 面向切面编程  

![image-20221220130818509](spring.assets/image-20221220130818509.png)

为了让程序员在编程时注重本身的核心业务逻辑，不会去在一个业务中交叉着另外一个业务处理。  

#### 总结

将与核心业务不相关的==交叉业务[^1]== 独立的抽取出来，然后以横向交叉的方式应用到业务流程中的过程就叫做AOP,面向切面编程

[^1]:  如安全模块，事务处理模块等等大量逻辑重复，通用的被称为==交叉业务== 。 

##### AOP优点

- 代码复用性增强
- 代码更易维护
- 开发者更关注业务逻辑

#### AOP七大术语  

- 连接点 JointPoint  
- 切点 JointCut  
- 通知 Advice
     - 前置通知@Before
     - 后置通知@AfterReturning
     - 环绕通知@Around
     - 异常通知@
     - 最终通知@After


- 切面(连接点+切点) Aspect  
- 织入点 Weaving  
- 目标对象 Proxy  
- 代理对象 Target  

#### 切点表达式

```java
execution([访问控制权限修饰符] 返回值类型 [全限定类名]方法名(形式参数列表) [异常])
```

访问控制权限修饰符：

- 可选项。
- 没写，就是4个权限都包括。
- 写public就表示只包括公开的方法。

返回值类型：

- 必填项。
- \* 表示返回值类型任意。

全限定类名：

- 可选项。
- 两个点“..”代表当前包以及子包下的所有类。
- 省略时表示所有的类。

方法名：

- 必填项。
- *表示所有方法。
- set*表示所有的set方法。

形式参数列表：

- 必填项

- () 表示没有参数的方法
- (..) 参数类型和个数随意的方法
- (*) 只有一个参数的方法
- (*, String) 第一个参数类型随意，第二个参数是String的。

异常：

- 可选项。
- 省略时表示任意异常类型。



# spring对事务的支持

事务(transaction)  



- 事务的四个特性
     - 原子性：事务时最小的工作单元，不可再分
     - 一致性：事务要么同时成功或者同时失败。事务前和事务后的总量不变
     - 隔离性：事务和事务之间有隔离性，才可以保证互不干扰
     - 持久性：持久性是事务结束的标志