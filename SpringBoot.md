## Spring

### Spring Springboot SpringCloud的区别

Spring 一整套的技术体系 包括Spring FrameWork Spring Data SpringSecurity Spring Boot SpringCloud都属于Spring大家族

SpringMVC是Spring基础上的一个MVC框架，主要处理WEB开发的路径映射和视图渲染

SpringFrameWork 是一个轻量级的java开发框架，核心是IOC控制反转（管理对象的生命周期）和AOP（面向切面编程）将主线业务和一些功能性代码分离

Spring Boot 整合了第三方框架，使用默认大于配置的方式，简化配置流程，内置http服务，命令方式启动项目

SpringCloud是基于Springboot的一整套的微服务的落地实现，包括配置管理，注册中心，服务发现，网关，熔断器等。

### Spring 核心

Spring Core 核心类库，提供IOC服务

Spring Aop： AOP服务

Spring ORM

Spring MVC

Spring Web

Spring DAO

### Spring4.X 新特性

```java
1、@RestController = @ResponseBody + @Contrroller
    而且添加了一个AsyncRestTemplate，支持REST客户端的异步无阻塞支持.

```

### SpringMVC

#### SpringMVC组件组成

```java
DispatcherServlet：作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展性。

HandlerMapping：通过扩展处理器映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。 

HandlAdapter：通过扩展处理器适配器，支持更多类型的处理器。

ViewResolver：通过扩展视图解析器，支持更多类型的视图解析，例如：jsp、freemarker、pdf、excel等。
    
dispatcherServlet 前端控制器 统一调度
handlerMapping 映射器 找到对应的handler
handlAdapter 映射器 适配器 执行对应的handler 返回 ModelAndView    
viewResolver 视图解析器    
```

#### SpringMVC的执行流程

 ![img](https://images2015.cnblogs.com/blog/249993/201612/249993-20161212142542042-2117679195.jpg) 

```java
1.用户向服务端发送一次请求，请求先到前端控制器（dispatcherServlet 中央控制球）
2.dispatcherServlet接收到请求后会调用HandlerMapping处理器映射器，得知该请求由那个Controller执行
3.dispatcherServlet代用HandlerAdapter处理器适配器，告诉适配器应该要去执行那个Controller
4.HandlerAdapter处理器适配器执行Contoller并且得到ModelAndView(数据和视图)，返回给dispatcherServlet
5.dispatcherServlet将ModelAndView交给ViewReslove视图解析器解析，然后返回真正的视图。    
```

### Spring AOP

```java
Aop:面向切面编程，切面：将与业务无关的逻辑，但是对每个对象产生影响的公共行为，抽取并且封装为一个可重用的模块。可用于 日志 权限 事物处理 全局异常等
```

AOP的实现在于代理模式

代理分为：静态代理和动态代理，其中静态代理代表为AspectJ；动态代理为SpringAOP

AspectJ：AOP框架会在编译阶段生成AOP代理类，因此成为编译增强，他会在**编译阶段将AspectJ（切面）织入到java字节码中**，运行的时候就是增强之后的AOP对象。

SpringAOP 动态代理，动态代理就是说AOP框架不会修改字节码，而是在**每次运行的时候临时为方法生成一个AOP对象**，这个AOP对象包含了目标对象的全部方法，并且在特定的切入点做了增强，并回调原对象的方法。

#### SpringAOP的动态代理

Spring Aop中使用 动态代理

1.JDK动态代理 基于接口

​	**只提供接口的代理，不支持类的代理**。核心为InvocationHandler接口和Proxy类。InvocationHandler通过invoke()方法反射调用目标类中的代码，动态的将横切逻辑和业务编织在一起；接着Proxy利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象	

​	总结： **Proxy.newProxyInstance生成一个目标类的代理对象**

​				newProxyInstance(args)中传入一个new InvocationHandler(){	//匿名内部类

​					//完成 横切逻辑和业务逻辑的编织

​				}

```java
//传递的soldier就是需要被增强的士兵类，
    public SoldierInterface MyProxyFactory(SoldierInterface soldier){
        //通知，此时的商店类，在工厂中new出的实列
        Store store = new Store();
        SoldierInterface proxysoldier = (SoldierInterface) Proxy.newProxyInstance(
                MyProxyFactory.class.getClassLoader(),//第一个参数：使用当前类的类加载器
                soldier.getClass().getInterfaces(),//第二个参数：传递的soldier士兵对象的接口
                new InvocationHandler() {//第三个参数，匿名内部类形式
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) 			throws Throwable {
                        //前置调用商店的配置
                        store.Before();
                        //调用目标方法，soldier就是传递的士兵对象，args参数默认无参
                        Object invoke = method.invoke(soldier, args);
                        //后置，士兵执行完方法后调用
                        store.After();
                        return invoke;
                    }
                }
        );
        return proxysoldier;
    }
```



2.CGLIB动态代理

​	没有实现InvocationaHandler接口，则使用CGLIB动态代理目标类，**通过继承的方式生成动态代理，如果某个类设置为final，则不可以使用CGLIB完成动态代理**。

​	总结:

​		1.在CGLIB代理类中 创建 Enhancer 对象。

​		2.enhance.setSuperClass(目标类.getClass) 继承目标类。

​		3.enhance设置回调函数，完成目标类的增强。

```java
public BowsSoldier MyCglibProxyFactory(BowsSoldier solider){
        //创建通知类
        Store store = new Store();
        //创建Enhance对象
        Enhancer enhancer = new Enhancer();
        //设置弓箭兵类对象
        enhancer.setSuperclass(solider.getClass());
        //设置回调方法
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                //放在方法执行前，就是前置增强，装备弓箭
                store.Before();
                //执行方法，solider就是传递过来的弓箭兵对象
                Object invoke = method.invoke(solider);
                //后置增强
                store.After();
                return invoke;
            }
        });
        //创建装备好的士兵
        BowsSoldier target1= (BowsSoldier) enhancer.create();
        return target1;
    } 
```

### Spring 创建bean的方式

1.调用构造函数创建Bean					

2.调用静态工厂方法创建Bean 

3.调用实例工厂方法创建Bean

4.Springboot注解 @Compent  @Configuration +@Bean   @Configuration+@Bean(工厂类)+@Bean(返回具体bean)

```java 
使用工厂类创建bean
    public class DemoBean{	//类
	}
	public class DemoBeanFactory implements FactoryBean<DemoBean>{	//工厂类
        public DemoBean getObject(){
            
        }
        public Class<?> getObjectType(){			
            return DemoBean.class
        }
        public boolean isSingleton(){
            return true
        }
    }

	@Configuration
	public class BeanLodingFactory{
        @Bean
        public DemoBean demoBean(){
            return demoBean;
        }
        @Bean
        public DemoBean getDemoBean(DemoBean demoBean){
            return demoBean.getObject()
        } 
        
        
    }
    
```

### Spring IOC

```java
IOC:依赖注入在初始化的过程中就不可避免的会写大量的new。这里IOC容器就解决了这个问题。这个容器可以自动对你的代码进行初始化，你只需要维护一个Configuration（可以是xml可以是一段代码），而不用每次初始化一辆车都要亲手去写那一大段初始化的代码。
所谓的依赖注入，则是，甲方开放接口，在它需要的时候，能够讲乙方传递进来(注入)
所谓的控制反转，甲乙双方不相互依赖，交易活动的进行不依赖于甲乙任何一方，整个活动的进行由第三方负责管理。
```

Spring让对象的创建不用去new，而是通过Spring容器自动生成，使用java的反射机制，根据配置文件在运行时动态创建以及管理。

IOC的三种注入方式：1.构造器注入  2.setter方法注入  3.注解注入 (@AutoWired+@Qualifier/@Resource)

```java
1、@AutoWired直接注入 有可能会 npe 空指针异常 (reqeust = true)
2、@set函数注入
    public void setXxx(Xxx xxx){
    	this.xxx = xxx
	}
3、构造器注入
     public AmsFileServiceImpl(AmsFileRepository amsFileRepository) {
        this.amsFileRepository = amsFileRepository;
    }
```

#### SpringIOC的优势

```java
是解耦的，平时new A()时候是要import包地址的，这就已经写死了，以后这个引用就死死的指向了那个类，想改变很麻烦，用ac.getbean(“A”)就没引入包，也就是所谓的不依赖 （就是跟那类A没关系），它只从容器找那个叫A的类，至于你给我的是啥，看容器中咋配置。举了例子：比如说是个很常用的dao类，开始你new的很开心，万一以后需求大改，数据库mysql换db2了，这个dao文件基本就得重写，如果这个类已经封装编译为class文件，不能改了怎么办。又或者你实例化了一个常用接口。
    我们不创建对象，通过容器创建管理依赖
```



### AutoWired和Resource的区别

1.@Autowired 按照type注入，当有重名的bean 使用@Qualifier("name"）来区分

```java
1.一个接口只有一个实现的情况下，属性名字怎么写都无所谓，因为按照类型匹配就只有一个bean
2.一个接口多个实现的情况下：
	① 属性名字跟组件名字一致，组件名字可以在声明的时候指定，比如 @Service("abc")
    ② 属性名字跟组件名字不一致，配合@Qualifier 注解指定组件名字

```

2.@Resource默认按照name注入，属于java的注解

### @Value 读取配置文件中的值

```java
1.非静态属性 直接使用@Value("${name}")读取
2.静态属性 	的赋值 
	private static String testdAd;

	@Value("${testdAd}")
	public void setTestdAd(String testdAd){
		AmsFileTask.testdAd = testdAd;
	}
    
```

### @ConfigurationProperties读取配置文件

```java
1.resources下创建 config.properties文件
    	demo.phone=10086
		demo.wife=self
2. 创建@Component引用配置文件中的属性  @ConfigurationProperties
@Component
@ConfigurationProperties(prefix = "demo")
@PropertySource(value = "config.properties")
public class ConfigBeanProp {
 
    private String phone;
 
    private String wife;
 
    public String getPhone() {
        return phone;
    }
 
    public void setPhone(String phone) {
        this.phone = phone;
    }
 
    public String getWife() {
        return wife;
    }
 
    public void setWife(String wife) {
        this.wife = wife;
    }
} 
```

### Springboot的过滤器和拦截器

```java
1.过滤器是在请求进入容器后，但请求进入servlet之前进行预处理的。请求结束返回也是，是在servlet处理完后，返回给前端之前
2.拦截器是spring提供并管理的，spring的功能可以被拦截器使用，在拦截器里注入一个service，可以调用业务逻辑。而过滤器是JavaEE标准，只需依赖servlet api ，不需要依赖spring。   
```

```java
定义过滤器
    @Component
	// 定义filterName 和过滤的url
	@WebFilter(filterName = "my2Filter" ,urlPatterns = "/*")
定义拦截器
    1.定义
   	MyInterceptor implements HandlerInterceptor
    2.引入
    @Configuration
	public class InterceptorConfig implements WebMvcConfigurer {
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new MyInterceptor());
        }
	}
执行顺序
    1.先过滤器
    2.在拦截器
    
```

### Springboot 读取配置文件的顺序

```java
1.执行命令下新建 config文件夹（jar包同一个目录下新建config文件夹），外部配置文件放在这个文件夹下，执行命令需在在jar包所在的目录下。 也可以使用 spring.config.location=config文件目录
2.读取classPath下congfig文件夹中的配置
3.最后读取src/main/resource文件下创建的application.properties（优先级最低）    
总结：
    先加载 jar包外的配置文件	在读取jar包内的；先读取 profile的	在读取不带profile的
BootStrap.yml 和 application.yml配置文件的加载顺序
    bootstrap.yml先加载，application.yml再加载
    bootstrap.yml属于系统级别的配置参数，加载之后内容不会被覆盖，application.yml中的内容不会覆盖bootstrap.yml中的内容
    
```



#### BeanFactory 和ApplicationContext的区别

1.BeanFactory是Spring 最底层的接口，包含各种Bean的定义，管理bean的加载，实例化，控制bean的生命周期，维护bean之间的依赖关系。

1.BeanFactory采用的是延迟加载形式注入bean，只有在使用到某个bean的时候，才会对该bean进行加载实例化。

2.ApplicationContext接口作为BeanFactory的派生类，除了含有BeanFactory的功能外，还提供了更加完整的框架功能，支持国际化，统一的资源文件访问方式，同时加载多个配置文件，载入多个上下文

2.ApplicationContext在容器启动时，一次性创建所有的bean。当配置bean过多的时候，可能会造成程序启动过慢。

### BeanFactory和FactoryBean

BeanFactory是Spring最底层的接口，包含各种bean的定义，实例化bean，维护bean之间的依赖关系

FactoryBean是一个工厂bean，可以根据类型返回不同的bean。

```java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    @Nullable
    T getObject() throws Exception;

    @Nullable
    Class<?> getObjectType();

    default boolean isSingleton() {
        return true;
    }
}
```

### Spring中的bean的作用域

1.singleton 默认  单例 每个容器只有一个bean的实例，单例的模式有BeanFactroy自身维护

2.request

3.session

4.global-session

5.prototype 为每一个bean请求创建一个实例

#### Springbean是线程安全的吗

```java
单例bean是线程不安全的，所有的线程都共享一个单例bean，因此存在资源竞争；如果bean是无状态的，也就是线程的操作不会对bean的成员执行查询之外的操作，那么这个bean就是安全的，比如 controller service dao这些都是无状态的。
解决方法：
    1.scope(prototype)
    2.使用同步机制 synchronized  或者 lock
    3.threadlocal
   
```



### Springbean 单例和多例的注入

```java
个单例beanA注入一个多例beanB时（多例注入单例是没有问题的），仅仅通过配置Spring是无法帮我们的单例beanA注入一个多例beanB的，即无法让我们每次使用beanB时都使用的是一个全新的beanB。因为beanA只初始化一次，相对应的Spring只会给beanA注入一个beanB。
解决办法是给beanA注入一个BeanFactory，这样我们就可以在每次需要使用beanB时都从BeanFactory中获取一个新的beanB。
```

### SpringBean重名

```java
1.如果重名，会抛出异常
    使用@Component("name")进行一个规避
2.两个第三方服务我都得依赖，但是他们有同名的bean
    在启动类添加注解@ComponentScan并指定其中的excludeFilters属性
@SpringBootApplication
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SameA.class)})
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### Spring 创建bean的流程

1.实例化

2.属性赋值

3.初始化

4.销毁

 ![img](https://upload-images.jianshu.io/upload_images/4558491-dc3eebbd1d6c65f4.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

1.getBean(beanName)，向容器请求某一个bean时，如果注册了 InstantiationAwareBeanPostProcessor ，在实例化之前先执行 postProccessBeforeInstantiation 

2.根据不同的配置Bean 构造函数或工厂方法实例化 Bean

3.如果注册了InstantiationAwareBeanPostProcessor ，在实例化对象之后，执postProcessAfterInstantiation 

4.Bean的属性赋值

5.执行afterPropertiesSet()

6.init- method 

7.如果在 <bean> 中指定 Bean 的作用范围为 `scope="prototype"`，则将 Bean 返回给调用者，由调用者负责 Bean 后续生命的管理 。 如果作用范围设置为 `scope="singleton"` ，则将 Bean 放入到 Spring   IoC 容器的缓存池中，并将 Bean 的引用返回给调用者， Spring 继续对这些 Bean 进行后续的生命管理。

#### Spring @PostContruct

Construct  -> Autowired -> @PostConstruct

#### Spring解决 循环依赖

1. 构造器的循环依赖，没有解决

2. setter函数注入（单例）依赖注入，解决

   案例：

   ​	A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”这种循环依赖的情况。

   ​	1、A首先完成了初始化的第一步，并且将自己提前曝光到**singletonFactories**中，

   ​	2、此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存**singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象**(虽然A还没有初始化完全，但是总比没有好呀)，B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。

   ​	3、此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化。
   

### Spring框架中使用到的设计模式

1.工厂模式 BeanFactory FactoryBean

2.单例模式 Bean的创建默认为单例

3.代理模式 SpringAOP使用到的 JDK动态代理 和CGLIB动态代理

4.模板方法：用来解决代码重复的问题，RestTempalate 	JpaTemplate	JmsTemplate  jdbcTemplate

5.原型模式： @Scope("prototype")  为每一个bean请求创建一个实例

5.享元模式线程池 和 String常量池。

### Spring 事务

```java
事物的特性：ACID
    原子性
    一致性
    隔离性
    持久性
Spring事物的配置 方式
    1.编程式事务管理
    	TransactionTemplate
    2.声明式事务管理
    	@Transactional注解	无法像编程式事务作用到代码块上事物的传播机制

    @Transaction()的接口
     Propagation propagation() default Propagation.REQUIRED;	//传播机制

     Isolation isolation() default Isolation.DEFAULT;			//隔离级别

	 boolean readOnly() default false;
	
	 int timeout() default -1;									//事务的超时机制
		
     Class<? extends Throwable>[] rollbackFor() default {};		//默认 RunTimeException 对于 checkException不会回滚


```

#### 事务的隔离级别

```java
1.Spring事务的默认
    Isolation.DEFAULT  使用数据库本身使用的 oracle 是 RC  mysql是 RR
互联网中使用
    读已提交 RC 作为事务隔离级别
    原因：在RR级别 条件未命中索引会锁表，而在RC隔离级别 只锁行
    update test set color = 'blue' where color = 'red'; 
    
```

#### 事务的传播机制

```java
1.REQUIRED 默认的属性，如果存在一个事物，则支持当前事物；如果没有事物则会开启一个事物
2.Mandatory 支持当前事物，如果当前没有事物，就抛出异常
3.Never 不支持事物
4.REQUIRES_NEW  新建事务，如果当前事物存在，就把当前事物挂起
    使用场景：需要在一个方法内部提前提交一部分事务或者是让内部的一段代码处于单独的一个事务管理的时候需要用到REQUIRES_NEW
5.supports	支持当前事物，如果没有就以非事务方式执行 （有就用 没有就没有）
    
说明：    在一个Service内部，事务方法之间的嵌套调用，普通方法和事务方法之间的嵌套调用，都不会开启新的事务.
A() 没有事务 B()有事务    A里面调用B 是不会开启新的事务的
    解决方法：这两个方法分开到不同的类中
```

#### 事务的传播机制的说明 REQUIRES_NEW

```java
@transaction    
public void testMethod(){
     dosomethingBefore...
     methodA();
     dosomgthingAfter...
} 

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodA(){
     updateSomething();
} 
Spring会帮我们开启一个事务
运行到methodA()这行的时候会挂起当前事务，然后重新创建一个事务，在methodA()中没有发生异常的情况下，运行完methodA方法直接提交methodA的事务；后续再执行dosomgthingAfter。
总共会有4种情况：
    1.dosomethingBefore发生异常， testMethod()的事务回滚
    2.updateSomething发生异常，methodA()的事务回滚，testMethod()如果对这个异常进行了捕捉，testMethod()事务会正常提交
    3.updateSomething发生异常，methodA()事务回滚，testMethod()没有进行异常捕捉，也会回滚
    4.dosomethingAfer发生异常，只有testMethod()会进行回滚

```

### 事物的失效

```java
1.@Transactional 注解只能应用到 public 可见度的方法上，在protect private 上不起作用
2.Spring的AOP即声明式事务管理默认是对RuntimeException()异常或是其子类进行事务回滚；对于 checkException(IOExcepttion)不会捕捉， 使用@Transactional(rollbackFor=Exception.class)解决。
3.不带有事务的方法调用该类中带事务的方法，不会回滚。因为spring的回滚是用过代理模式生成的，如果是一个不带事务的方法调用该类的带事务的方法，直接通过this.xxx()调用，而不生成代理事务，所以事务不起作用。常见解决方法，拆类。
```



### 事务的说明

```java
1.@Transactional(rollbackFor=Exception.class)，如果类加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。
2.在@Transactional注解中如果不配置rollbackFor属性,那么事物只会在遇到RuntimeException的时候才会回滚,加上rollbackFor=Exception.class,可以让事物在遇到非运行时异常时也回滚    
```

#### Spring异常处理

```java
1.@ControllerAdvice 异常集中处理 
    控制器增强
    @ControllerAdvice注解后，这个GlobalExceptionHandle类就是个切面类，所有经过controller的方法的异常都能被他捕获
    其可以包含由@ExceptionHandler、@InitBinder 和@ModelAttribute标注的方法，可以处理多个Controller类，这样所有控制器的异常可以在一个地方进行处理。
2.@ExceptionHandler    
    对应异常类的处理
```



## Springboot

1、将原来的xml配置 简化为java注解

2、内容servlet容器 直接打成jar 运行

3、使用starter依赖自动完成bean配置 和引入相关jar包。

@EnabledAutoConfiguration完成 bean自动配置

相比spring FrameWork的优点

1.快速构建项目

2.内置web容器

3.自动配置，自动管理依赖

4.自带应用监控 springboot acutotor

### 核心注解

1. @SpringBootApplication 包含 2 3 4
2. @SpringBootConfiguration 实现配置文件功能
3. @EnableAutoConfiguration 打开自动配置功能
4. @ComponentScan  组件扫描

### Springboot 自动配置原理

1.@EnableAutoConfiguration 自动配置

2.@Import(AutoConfigurationImportSelector.class)导入AutoConfigurationImportSelector类

3.SpringFactoriesLoader.loadFactoryNames()会加载类路径及所有jar包下META-INF/spring.factories配置中映射的自动配置类

Spring boot的自动配置原理

​	@EnableAutoConfiguration 找到 @import(AutoConfigurationImportSelector.class) ,通过loadFactoryNames加载 MATE-INF/spring.factories配置中所有的自定配置类到spring容器，每个自动配置类都是在某些条件下才会生效的

​	其中 @ConditionOnBean				 容器中有这个类,就注入这个类

​			@ConditionOnClass		

​			@ConditionOnMIssBean 		容器中没有这个bean 就注入

​			@ConditionOnProperty

其中以ServletWebFactoryAutoConfiguration类为例， 在该类上有@EnableCofigurationProperties(ServerProperties.class)主键，其中@EnableConfigurationProperties启用配置属性，后面的ServerProperties类

​	ServerProperties类上有注解 @ConfigurationProperties，将从配置文件中绑定属性到对应的bean上。

在全局的配置属性中 如： server.port 首先通过@ConfigurationPorperties注解绑定到 xxxProperties实体类，封装为一个bean，再通过@EnableConfigurationProperties注解导入到Spring容器中。

#### Springboot自动配置的关键

1. @Configuration @Bean  基于java代码的bean配置

2. @Conditional  设置自动配置条件依赖

3. @EnableConfigurationProperties @ConfigurationProperties 将配置文件和bean属性绑定

4. @EnableAutoConfiguration @Import 实现bean的加载和发现

   流程：

   ​	Springboot 启动类 @SpringBootApplication 中 的@EableAutoConfiguration 通过 @import(), SpringFactoriesLoader.loadFactoryNames找到META-INF/Spring.factories配置文件，加载所有的自动配置类到spring容器，每一个类都有自己特定的特定生效条件 @ConditionXx注解， 通过@EableConfigurationProperties启用配置属性 ，根据XXProperties类上的@ConfigurationProperties注解将配置文件中的值和类属性绑定，封装成bean。

#### Springboot 内置tomcat配置

1.server.port 端口

2.server.address 地址

3.server.tomcat.access.log.enabled  开启tomcat日志

4.server.tomcat.access.log.directory  tomcat日志输出地址

5.server.tomcat.max-threads  200 最大并发数

6.server.tomcat.max-connections  10000 最大连接数

### Springboot启动的流程

```java
@SpringBootApplication包括三个注解：
@EnableAutoConfiguration：SpringBoot根据应用所声明的依赖来对Spring框架进行自动配置。简单概括一下就是，是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器。

@Configuration：它就是JavaConfig形式的Spring Ioc容器的配置类。被标注的类等于在spring的XML配置文件中(applicationContext.xml)，装配所有bean事务，提供了一个spring的上下文环境。

@ComponentScan：组件扫描，可自动发现和装配Bean，功能其实就是自动扫描并加载符合条件的组件或者bean定义，最终将这些bean定义加载到IoC容器中。可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。默认扫描SpringApplication的run方法里的Booter.class所在的包路径下文件，所以最好将该启动类放到根包路径下

具体的启动流程：
    1.运行run方法之前需要对SpringApplication进行初始化，会调用一个private类型的的initialize方法
    	1.1sources设置到SpringApplication属性
    	1.2判断是否为web程序，并设置到webEnvironment这个boolean属性中。
    	1.3找出所有的初始化器以及应用监听器，默认有5个，设置到initializers属性中 。
    2.调用run方法，启动SpringApplication
    	2.1构造一个StopWatch，观察SpringApplication的执行
    	2.2找出所有的SpringApplicationRunListener并封装到SpringApplicationRunListeners中，用于监听run方法的执行。监听的过程中会封装成事件并广播出去让初始化过程中产生的应用程序监听器进行监听 
    3.构造Spring容器(ApplicationContext)。
    	3.1Spring容器的判断是否是web环境，是的话AnnotationConfigEmbeddedWebApplicationContext，否则构造 AnnotationConfigApplicationContext
    	3.2初始化spring容器
    	3.3最核心的一步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的ApplicationContext
    	3.4A调用ApplicationContext的refresh()方法
    	
```



### Springboot 自定义异常 配置全局异常处理

##### RestControllerAdvice

使用全局异常处理器 `@RestControllerAdvice` 处理请求发送的异常

- @RestControllerAdvice：默认会扫描指定包中所有**@RequestMapping**注解

  @ExceptionHandler、@InitBinder、@ModelAttribute，并应用到所有@RequestMapping

  

```java
自定义全局异常处理类
    1.@RestControlllerAdvice 注解 GlobalExceptionHandler类 
    @RestControlllerAdvice 注解对 controller进行增强
    2.@ExceptionHandler 配置的 value 指定需要拦截的异常类型，上面拦截了 Exception.class 这种异常。
   @RestControllerAdvice
public class GlobalExceptionHandler {
     
    @ExceptionHandler(Throwable.class)
    public ResponseEntity handleException(Throwable e) {
        // 打印堆栈信息
        log.error(ThrowableUtil.getStackTrace(e));
        return buildResponseEntity(ApiError.error(e.getMessage()));
    }
}
    
    
```

![image-20200313180647161](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200313180647161.png)

### Springboot 使用redis作为缓存

1.springboot默认使用 simpleCache作为缓存，创建map的形式

2.引入 redis的starter

3.重写redisTemplate的 <K,V>的序列化

4.key使用 StringRedisSerializer

5.value使用 FastJsonRedisSerializer



#### SpringBoot配置多数据源

```java
1.启动类关闭Spring Boot下DataSource 的JPA自动配置
    @SpringBootApplication(exclude = {
        DataSourceAutoConfiguration.class,
        HibernateJpaAutoConfiguration.class,
        DataSourceTransactionManagerAutoConfiguration.class})
@EnableTransactionManagement
public class ManControllerExclude {

    public static void main(String[] args) {
        SpringApplication.run(ManControllerExclude.class,args);
    }
}

1.配置文件
    dataSource:
		dataSources1:
			url:jdbc database1
		dataSources2:
			url:jdbc database2
2.@ConfigurationProperties(prefix = "spring.datasource.database1")
@Component
@Data
public class DataBase1Properties {
    private String url;

    private String username;

    private String password;

    private String driverClassName;
}
            
@ConfigurationProperties(prefix = "spring.datasource.database2")
@Component
@Data
public class DataBase2Properties {
    private String url;

    private String username;

    private String password;

    private String driverClassName;
}

3.@Configuration
@Slf4j
public class DataSourceConfig {
    @Autowired
    private DataBase1Properties dataBase1Properties;
    
    @Autowired
    private DataBase2Properties dataBase2Properties;
    
    @Bean(name = "dataBase1DataSource")
    @Primary
    public DataSource dataBase1DataSource(){
        log.info("dataBase1DataSource初始化----111111");
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(dataBase1Properties.getUrl());
        dataSource.setUsername(dataBase1Properties.getUsername());
        dataSource.setPassword(dataBase1Properties.getPassword());
        dataSource.setDriverClassName(dataBase1Properties.getDriverClassName());
        return dataSource;
    }

    @Bean(name = "dataBase2DataSource")
    public DataSource dataBase2DataSource(){
        log.info("dataBase2DataSource初始化----222222");
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(dataBase2Properties.getUrl());
        dataSource.setUsername(dataBase2Properties.getUsername());
        dataSource.setPassword(dataBase2Properties.getPassword());
        dataSource.setDriverClassName(dataBase2Properties.getDriverClassName());
        return dataSource;
    }
}

4.配置工厂类和ordersEntityManagerFactory
    @EnableJpaRepositories的作用是开启Jpa的支持,因为我们会有多个自定义JPA，就需要单独实现各自的管理类，其中，entityManagerFactoryRef是实体关联管理工厂类和transactionManagerRef事务管理类，都需要我们自行实现。
1：Spring 3中引入了属性管理类，这里主要用来引入一些系统环境属性。
2：定义前缀，并返回DataSourceProperties对象，为什么？这是因为自动配置时加载的对象DataSourceAutoConfiguration会定义它，并通过它读取配置的内容，这里new个新对象就行了，主要是为后面的bean初始化服务。
3：获取前缀的DataSourceProperties对象，并创建真正的DataSource数据源,这里我们使用的是Spring Boot自带的工具类DataSourceBuilder，值来源的就是从前缀对象中读取的值，换句话说，就是在配置文件里我们写的值；
4：事物管理器的主接口PlatformTransactionManager需要获取到JpaTransactionManager的对象进行事务管理，这个对象就是由下面//5的工厂方法创建的。

```

### json的顺序 序列化文件的作用  redis set为null 怎么排查

