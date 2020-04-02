## java基础知识点

### java的三大特性

封装

​	Class类  将不需要的代码隐藏，将公有方法进行提供对外访问

继承

​	extends 继承父类的非private函数 以及非private属性

多态

​	一种方法可以有多个版本的实现

​	方法的重载和重写

### Object的函数

```java
clone  toString	hashCode	equals	getClass	wait	noify	notifyAll	finalize
```

### public protect default private 

|           | **同一个类** | **同一个包** | **不同包的子类** | **不同包的非子类** |
| --------- | ------------ | ------------ | ---------------- | ------------------ |
| Private   | √            |              |                  |                    |
| Default   | √            | √            |                  |                    |
| Protected | √            | √            | √                |                    |
| Public    | √            | √            | √                | √                  |

**public： Java语言中访问限制最宽的修饰符，一般称之为“公共的”。被其修饰的类、属性以及方法不
　　　　　仅可以跨类访问，而且允许跨包（package）访问。
private: Java语言中对访问权限限制的最窄的修饰符，一般称之为“私有的”。被其修饰的类、属性以
　　　　　及方法只能被该类的对象访问，其子类不能访问，更不能允许跨包访问。
protect: 介于public 和 private 之间的一种访问修饰符，一般称之为“保护形”。被其修饰的类、
　　　　　属性以及方法只能被类本身的方法及子类访问，即使子类在不同的包中也可以访问。
default：即不加任何访问修饰符，通常称为“默认访问模式“。该模式下，只允许在同一个包中进行访**

总结：

​	public 全局都可以访问

​	private 只有本类可以访问

​	protect 本类 本包 不同包的子类

​	default 本类 本包

​	只有public 和default 可以修改 外部类

### 重载和重写

重写是子类对父类允许访问的方法（子类有访问权限）的实现过程进行重新编写，返回值和参数都不能改变

​	1.final的函数不可以被重写

​	2.static的方法不可以被重写

​	3.构造函数不可以被重写，因为构造函数式私有的，并不能被继承，所以就不会被重写。

重载是在一个类里面，方法名字相同，参数不同，**返回的类型可以相同也可以不同** 	**重载必须在同一个类**。

​	1.被重载的方法必须改变参数列表

​	2.被重载的方法可以改变 访问修饰符

​	3.构造函数可以被重载

### 接口和抽象类

```java
接口中的方法都是没有方法体的，抽象类中可以有 具有方法体的方法
一个类可以实现多个接口，但是只可以实现一个抽象类
接口可以继承extends接口，抽象类可以实现implement接口
接口中的方法是public的，抽象类中的方法可以是public protectd default
```

### Static关键字

```java
1.修饰静态变量，也叫类变量，直接可以使用类去调用，不用实例对象，并且静态变量只存在一份。
    static 不可以和 abstract同时修饰
```



### String StringBuildre StringBuffer

```java
String : final char[] 是不可变的
StringBuidler 线程不安全的 对StringBuilder对象本身进行操作   没有final修饰
    操作的是StirngBUilder对象
StringBuffer  线程安全的，加入了同步锁。    
```

### 在Java中定义一个 无参构造函数

Java程序在执行子类的构造函数之前，如果没有使用super()来调用父类特定的构造函数，就会调用父类中“无参构造函数”。因此如果类中没有无参构造函数，就会在编译时发生错误。

### 抽象类和接口的异同点

相同：	

​	都不可以被实例化，因为抽象类中有未实现的抽象方法，实例化的时候不能正常分配内存；

​	 抽象类里的抽象方法必须全部被子类所实现，如果子类不能全部实现父类抽象方法，那么该子类只能是抽象类。同样，一个实现接口的时候，如不能全部实现接口方法，那么该类也只能为抽象类。 

不同点：

​	抽象类被extends 接口被implement

​	抽象类可以有函数体的函数

​	 如果一个类里有抽象方法，那么这个类只能是抽象类 

### 线程的状态

```java
New 		初始状态，线程被构建，还没没有调用start()
Runnable	运行状态
blocked		阻塞状态	thread.yied() 该线程就会阻塞，等待其他线程执行完毕    
waiting		等待状态	需要其他线程 notify notifyAll 进行唤醒
terminated	终止状态    
```

### final

可以修饰 变量 方法 类

```java
修改变量：如果修改基本类型变量，在数值初始化之后，是不可以更改；如果修饰引用类型变量，初始化后就不可以再指向其他对象
修饰类：该类不可以被继承，该类中的所有成员方法都被隐式的修饰为final
修饰方法：锁定函数，防止被修改，类中所有的privaye都被隐式的指定为final
    
```

### 什么时候finally不会被执行

```java
1.finally语句块第一行发生了异常
2.System.exit()已经退出程序
3.关闭CPU
4.线程直接kill
```

### java IO

```java
基于字节
    InputStream
    OutputStream
    	FileOutputStream
    	PipedOutputStream
    	ByteArrayPutputStream
基于字符
   	Reader
    	FileReader
    	PipedReader
    	CharArrayReader
    	InputStreamReader
    	BufferedReader
    Writer
    	
```

### java NIO

```java
同步非阻塞IO
    Channle
    Selector
    Buffer
NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。 
```

 ![img](https://img-blog.csdn.net/20180905114503963?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM4NTc0NTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 



### java8新特性

#### 接口中可以含有默认函数

```
1.接口中可以添加一个非抽象方法，使用default修饰，实现这个接口的类可以直接调用该default函数
```

#### 集合的stream

#### Lambda

#### Optional

### forward 和 redirect

1.forward 请求转发

```java
request.getRequestDispatcher("new.jsp").forward(request, response);   //转发到new.jsp
```

2.redirect 请求重定向

```java
response.sendRedirect("new.jsp");   //重定向到new.jsp
```

区别

```java
1.forward 由request对象调用
    		可以共享request中的数据
    		服务器直接进行转发,浏览器url地址不改变
  redicrect response对象调用   
    		服务端根据逻辑,发送一个302状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL，将要重定向的url 
    		String location = con.getHeaderField("Location");
         	url = location;
    		重定向行为是浏览器做了至少两次的访问请求的。
    		
    		
```

### RESTFul

```java
1. 基于“资源”，数据也好、服务也好，在RESTFul设计里一切都是资源。 url 代表资源
2. 无状态。一次调用一般就会返回结果，不存在类似于“打开连接-访问数据-关闭连接”这种依赖于上一次调用的情况。
3. URL中通常不出现动词，只有名词
4. URL语义清晰、明确
5. 使用HTTP的GET、POST、DELETE、PUT来表示对于资源的增删改查
6.返回不同的状态码 200，201,202,204,400,401,403,500。比如401表示用户身份认证失败，403表示你验证身份通过了，但这个资源你不能操作。
    GET -  SELECT
    POST - CREATE 
    PUT  - UPDATE  
    DELETE - DELETE 
POST 和 PUT的 区别
    PUT是幂等的操作，即重复操作不会产生变化，10次PUT 的创建请求与1次PUT 的创建请求相同，只会创建一个资源，其实后面9次的请求只是对已创建资源的更新，且更新内容与原内容相同，所以不会产生变化。
	POST 的重复操作截然不同，10次POST请求将会创建10个资源。
    

```

### java异常

 ![img](https://img-blog.csdn.net/20160326233035366) 

```java
uncheckedException 和 checkedException
    uncheckedException : 非限制异常 包括 Error + RuntimeException 派生类，是在程序运行期间发生的不可限制的异常。
    checkedException:	 限制异常   IOException 创建的时候，就需要使用 try{ } catch(){} 进行异常捕获；或者使用 throws Exception 往上级方法抛出   
     	
```

### MAVEN

```java
clean:项目清理，删除target下编译的内容
complile 项目编译
test:项目运行测试
package 打包
install 打包 + 打包的结果放在本地仓库，可供其他项目或者模板引用
```





