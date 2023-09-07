# 语法

## map和object的区别

1. map是干净的只含有显示插入的键,而普通Object上会有原型上的属性以及方法
2. es5之后可以说使用Object.create(null)来创建一个干净的对象
3. map的键可以是任意的数据类型包括基本的数据类型对象以及函数,而object只允许使用symbol以及string
4. map中的key是有序的迭代的时候以其插入的顺序返回键值而object的键是无序的
5. map长度可以通过size方法来获取而object需要手动计算(Object.keys(obj).length)
6. map是可迭代的object需要通过获取键来迭代
7. map在频繁增删键值对的场景下表现更好,object在频繁添加和删除键值对的场景下未作出优化。
## 注解

### 大全

[https://juejin.cn/post/7179038534711967800?searchId=20230727112322084EED6666EE0A91D11E#heading-11](https://juejin.cn/post/7179038534711967800?searchId=20230727112322084EED6666EE0A91D11E#heading-11)

### 理解

* @Bean
使用 @Bean 注解可以将方法返回的对象注册为一个 Bean，然后在其他地方就能通过@Autowrited进行注入并使用

* @Transactional
```java
public void yourMethod() {
    xxxMapper.insert();
    myClass zzh = new myClass();
    zzh.select();
}
// 加入insert向a表插入了一行，zzh类的select想要查找这一行，此时可能找不到，这很反直觉。
// 造成的原因：select并没有开启事务，且也没有加入yourMethod的事务，所以yourMethod事务中未提交的值select中看不见且数据库中也没有
@Transactional
public void yourMethod() {
    xxxMapper.insert();
    myClass zzh = new myClass();
    @Transactional(propagation = Propagation.REQUIRED)
    zzh.select();
}
```
1. propagation属性
propagation 代表事务的传播行为，默认值为 Propagation.REQUIRED，其他的属性信息如下：

    * Propagation.REQUIRED：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。( 也就是说如果A方法和B方法都添加了注解，在默认传播模式下，A方法内部调用B方法，会把两个方法的事务合并为一个事务 ）
    * Propagation.SUPPORTS：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
    * Propagation.MANDATORY：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
    * Propagation.REQUIRES_NEW：重新创建一个新的事务，如果当前存在事务，暂停当前的事务。( 当类A中的 a 方法用默认Propagation.REQUIRED模式，类B中的 b方法加上采用 Propagation.REQUIRES_NEW模式，然后在 a 方法中调用 b方法操作数据库，然而 a方法抛出异常后，b方法并没有进行回滚，因为Propagation.REQUIRES_NEW会暂停 a方法的事务 )
    * Propagation.NOT_SUPPORTED：以非事务的方式运行，如果当前存在事务，暂停当前的事务。
    * Propagation.NEVER：以非事务的方式运行，如果当前存在事务，则抛出异常。
    * Propagation.NESTED ：和 Propagation.REQUIRED 效果一样。
### 元注解

1. @Target用来规定自定义注解可以用来修饰什么东西
@Target({ElementType.METHOD})

```java
// @Target注解的枚举参数
public enum ElementType {

  	//类，接口，枚举类
    TYPE, 

  	//成员变量，枚举常量
    FIELD, 

	//方法
    METHOD,

	//形式参数
    PARAMETER,

 	//构造方法
    CONSTRUCTOR,

	//局部变量
    LOCAL_VARIABLE,

    //注解
    ANNOTATION_TYPE,

    //包
    PACKAGE,

    //类型参数
    TYPE_PARAMETER,

    //类型使用
    TYPE_USE
}
```
2. @Retention描述注解保留的时间范围
@Retention(RetentionPolicy.RUNTIME)

```java
// @Retention注解的枚举参数：
public enum RetentionPolicy {
   
   	//源文件保留
    //指定注解仅在源代码阶段保留，在编译后的字节码中将被丢弃。这意味着在运行时无法获取该注解。这种类型的注解通常用于提供给编译器或其他代码分析工具使用，而不会对程序运行产生影响。
    SOURCE,

  	//编译期保留，默认值
    //指定注解在编译时保留，并存储在生成的字节码文件中。但在运行时，无法通过反射获取该注解的信息。这是默认的保留策略，如果在 @Retention 中未指定 value 参数，默认为该值。
    CLASS,

  	//运行期保留，可通过反射去获取注解信息
    //指定注解在运行时保留，并可以通过反射机制在运行时获取和解析注解的信息。这种类型的注解通常用于运行时的动态处理。
    RUNTIME
}
//简而言之，RetentionPolicy.SOURCE 只在源代码阶段存在，RetentionPolicy.CLASS 在编译阶段存在但运行时无法获取，而 RetentionPolicy.RUNTIME 可以在运行时通过反射获取注解信息。
```
3. @Documented在使用javadoc工具为类生成帮助文档时是否要保留其注解信息。
@Documented

加上代表帮助文档中保留注解信息，反之不保留

4. @Inherited使被他修饰的注解具有继承性（如果某个类使用了被@Inherited修饰的注解，则其子类将自动具有该注解。）
@Inherited

```java
// 注解
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyInheritedAnnotation {
    public String name() default "pengjunlee";
}
// 父类
@MyInheritedAnnotation(name = "parent")
public class Parent {
}
// 子类
public class Child extends Parent {
    public static void main(String[] args) {
        Class<Child> child = Child.class;
        MyInheritedAnnotation annotation = child.getAnnotation(MyInheritedAnnotation.class);
        // 这里将输出父类上的注解的参数值name = "parent"
        System.out.println(annotation.name());
    }
}

```
5. @Repeatable用于声明标记的注解为可重复类型注解，可以在同一个地方多次使用。
@Repeatable(MyRepeatableAnnotation.class)

```java
// 第一个注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyRepeatableAnnotation {
    RepeatableAnnotationTest[] value();
}

// 第二个注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(MyRepeatableAnnotation.class)
public @interface RepeatableAnnotationTest {
    String key();
    String value();
}

// 使用
@RepeatableAnnotationTest(key = "aa",value = "11")
@RepeatableAnnotationTest(key = "bb",value = "22")
public class RepeatableTest {
    public static void main(String[] args) {
        RepeatableAnnotationTest[] annotation = RepeatableTest.class.getAnnotation(MyRepeatableAnnotation.class).value();
        for (RepeatableAnnotationTest a : annotation) {
            System.out.println(a);
            /*
            *输出
            *@annotation.RepeatableAnnotationTest(key=aa, value=11)
            *@annotation.RepeatableAnnotationTest(key=bb, value=22)
            */
        }
    }
}

```
6. @Native修饰成员变量，则表示这个变量可以被本地代码引用，常常被代码生成工具使用。对于@Native注解不常使用，了解即可。
### 自定义注解

* 定义注解。
```java
// 创建一个java文件并通过@interface来声明一个注解。创建t2.java文件
package com.example.oboqrcs.utils;
import java.lang.annotation.Target;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Target({ElementType.METHOD, ElementType.PARAMETER, ElementType.CONSTRUCTOR})
@Retention(RetentionPolicy.RUNTIME)
public @interface t2 {
    // 可以传入一个参数默认值是""
    String value() default "";
}
```
* 定义注解处理器。
```java
// 创建t3.java文件
package com.example.oboqrcs.utils;

import java.lang.reflect.Method;

public class t3 {
    public void beforeT2Annotation(Object obj) {
        // getClass()获得数据类型。输出：class com.example.oboqrcs.controller.t1$1zzhClass或者class java.lang.String类似
        Class clazz = obj.getClass();
        // getDeclaredMethods获取一个class中声明的method，返回Method []
        for (Method method : clazz.getDeclaredMethods()) {
            if (method.isAnnotationPresent(t2.class)) {
                System.out.println("=-=======here is t2 annotation===========");
                // 通过这里来获得传递给注解t2的参数
                t2 annotation = method.getAnnotation(t2.class);
                // 注意：这里是annotation.value()是因为t2参数名是value如果是zzh则变成annotation.zzh()
                String value = annotation.value();
                System.out.println("The value of @t2 annotation is: " + value);
            }
        }
    }
}

```
* 使用注解
```java
// 在t1.java中使用自定义的注解t2
import com.example.oboqrcs.utils.t3;
import com.example.oboqrcs.utils.t2;

class myClass {
  public myClass () {
      // 创建注解处理器实例
      t3 T3 = new t3();
      T3.beforeT2Annotation(this);
      // t3文件内容也不见得一定要封装之间搬到这里也能用
  }
  @t2("Hello World")
  public void testMethod () {
      // ToDo
  }
}
myClass zzh = new myClass();
```
* 总结
整个过程其实就是创建了两个class，然后在需要的class中使用它们。但是特别的是这两个新建的class不同于普通的class它们有各自的特点所以它们成为了注解。

t1.java中使用时需要先创建t3也就是注解处理器的实例然后还要调用它的方法，可以理解为启动了注解的钩子函数。

## 反射

反射：在运行状态中，对于任意一个类、对象，都能够知道这个类的所有属性和方法。

反射机制相关的重要的类

![图片](data:image/png;base64,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)


```java
// 注：必须先获得Class才能获取Method、Constructor、Field。
Class clazz = obj.getClass();
clazz.getDeclaredMethods()  // 返回Method []
```
## 双亲委派

双亲委派的目的就是让用户不能轻易的更改系统原本的类比如String、Integer这些。但是它并不会在自己定义String并使用时出现，具体出现的情况还有待明确

```javascript
class String {
    public Integer length () {
        return 1;
    }
}
String temp = new String();
System.out.println("===========length:" + temp.length());
System.out.println("===========classLoader:" + temp.getClass().getClassLoader());
// 1
// AppClassLoader
// 可以发现并不是双亲委派中的0和BootClassLoader。
// 有个说法是tomcat故意违背了双亲委派，但是我使用最原本的java项目最终还是一样的结果
```
## native

native关键字用于表示该方法的实现是由非Java代码提供的 。

在Java中，native方法通常用于以下情况：

1. 与底层操作系统进行交互，如系统调用；
2. 访问硬件设备或外部设备驱动程序；
3. 使用第三方库或语言（如C、C++）实现高性能和底层操作。
## JVM

### 堆(Heap)

主要用于存储Java对象和数组实例。它是Java程序运行时动态分配内存的区域。堆是JVM管理的最大的一块内存空间，也是垃圾回收（Garbage Collection，GC）的主要工作区域。

堆可以被划分为几个逻辑上的区域，其中两个主要的区域是：

1. 新生代（Young Generation）：新生代是堆的一部分，用于存放新创建的对象。在新生代中，又分为 Eden空间、Survivor 0空间和 Survivor 1空间。大部分对象在新生代中被创建，并且大多数对象都是短暂的，很快就会被回收。
* Eden空间：是对象创建的起始区域。当Eden空间满了之后，将触发Minor GC（新生代垃圾回收），清理掉不再被引用的对象，并将存活的对象复制到Survivor空间。
* Survivor空间：是存放从Eden空间中经过Minor GC后仍然存活的对象的区域。每次Minor GC之后，存活的对象会被复制到这个空间。两个Survivor空间中总有一个是空的。经过多次Minor GC后，仍然存活下来的对象会被移到年老代。
2. 年老代（Old Generation）：年老代用于存放经过一定时间生命周期较长的对象。这些对象在新生代中经历了一次或多次Minor GC后仍然存活下来，因此被认为是长时间存活的对象。大部分的业务逻辑代码、长期存活的对象以及大对象会被分配到年老代。
对象晋升：当Eden空间进行Minor GC后，如果某个对象经过多次回收后仍然存活，那么它会被晋升到年老代。

3. 元数据区域（Metaspace）:用于存放类加载器、类信息、常量池等元数据。这个区域是特别的，它不占用heap空间，通过：$$jvm运行时的heap的内存大小 = youngGen内存 + oldGen内存$$可以求证
### 性能分析

```javascript
jstat -gc <java项目的PID> 2000 5
// 表示2秒输出一次，一共输出5次
```
* S0C：年轻代中第一个survivor（幸存区）的容量 （字节）
* S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
* S0U ：年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
* S1U ：年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
* EC ：年轻代中Eden（伊甸园）的容量 (字节)
* EU ：年轻代中Eden（伊甸园）目前已使用空间 (字节)
* OC ：Old代的容量 (字节)
* OU ：Old代目前已使用空间 (字节)
* MC：metaspace(元空间)的容量 (字节)
* MU：metaspace(元空间)目前已使用空间 (字节)
* YGC ：从应用程序启动到采样时年轻代中gc次数
* YGCT ：从应用程序启动到采样时年轻代中gc所用时间(s)
* FGC ：从应用程序启动到采样时old代(全gc)gc次数
* FGCT ：从应用程序启动到采样时old代(全gc)gc所用时间(s)
* GCT：从应用程序启动到采样时gc用的总时间(s)
1. Eden区和Survivor区的使用量达到一定阈值时，将会触发一次Young GC（也称为YGC），这时会进行对象的垃圾回收。
2. Young GC过程中，存活的对象会被拷贝到空闲的Survivor区或者另一个Survivor区，同时清空Eden区和使用过的Survivor区。
3. 当对象在Survivor区经过多次复制后仍然存活，它们将会被晋升到老年代（Old Generation）。
4. 当Old Generation的空间使用率达到一定阈值时，将会触发一次Full GC。Full GC会对整个堆内存进行垃圾回收，包括Young Generation和Old Generation。
### gc log

在启动jar时可以使用某些命令来开启、配置gc log

* 老java版本命令
```java
java ^
‐Xloggc:./gc.log ^
‐XX:+PrintGCDetails ^
‐XX:+PrintGCDateStamps ^
‐XX:+PrintGCTimeStamps ^
-XX:+PrintGCApplicationStoppedTime ^
-XX:+PrintGCApplicationConcurrentTime ^
‐XX:+PrintGCCause ^
‐XX:+UseGCLogFileRotation ^
‐XX:NumberOfGCLogFiles=10 ^
‐XX:GCLogFileSize=100M ^
-jar main.jar
```

---
```java
java
‐Xloggc:./gc.log  // 日志文件带时间
‐XX:+PrintGCDetails  // 打印详细信息
‐XX:+PrintGCDateStamps  // 打印日期
‐XX:+PrintGCTimeStamps  // 打印时间
-XX:+PrintGCApplicationStoppedTime  // 打印 gc 停顿时长
-XX:+PrintGCApplicationConcurrentTime  // 打印 gc 间隔的服务运行时长
‐XX:+PrintGCCause  // 打印 GC 原因
‐XX:+UseGCLogFileRotation  // 开启日志轮换
‐XX:NumberOfGCLogFiles=10  // 日志保留个数
‐XX:GCLogFileSize=100M  // 每个日志文件的大小
main.jar
```
* 新java版本命令
```java
java -Xlog:gc*:./gc.log -jar oboqrcs-0.0.1-SNAPSHOT.jar
```
# 技巧

maven初始化项目前记得在conf/settings.xml下设置国内镜像

开发时遇到奇怪的问题解决不了，切换管理员身份试试

java中equals（）与 ==存在区别,==是比较两个变量的地址例如a=1，b=1，它们的地址并不相等所以a=b=>false，equals是比的变量的值，a.equals(b)=>true

java中interface与class的区别，interface多用于mapper中，还有其他？？

jsonobject类型用put（key， value）插入值，jsonarray用add（value），两个都是插在末尾

maven的pom.xml中第一句<?xml version="1.0" encoding="UTF-8"?>报错可以将此文件所有内容剪切再粘贴回来，可以解决报错，但是原理不明

```java
@Transactional(rollbackFor = Exception.class)
    public void functionA(JSONArray array) {
        for(int i = 0; i < array.size(); i++ ) {
            ProductDeliveryMapper.updataBookDelivery(array.get(i));
        }
    }

@Transactional(rollbackFor = Exception.class)
public void functionB(JSONArray array) {
    for(int i = 0; i < array.size(); i++ ) {
        ProductDeliveryMapper.updataBookDelivery(array.get(i));
    }
}
```
我有这两个方法在springboot中，如果现在正在执行functionA里面的for循环，现在后端程序又接受到访问functionB，那现在程序是等待functionA执行完毕再执行functionB，还是直接执行functionB使得functionA
答：在使用Spring事务管控时，如果两个方法同时被调用，那么默认情况下Spring会启动两个不同的事务来依次处理这些方法。也就是说，如果当前正在执行 functionA 的事务未提交，那么将等待其提交后才能开始执行 functionB 的事务。 但是可以写代码改变事务的处理逻辑

# 解决方案

## java版本切换

1. 环境变量修改
2. C:\Program Files (x86)\Common Files下可能存在Oracle文件夹，改名或者删除
3. setting.json中修改
## jwt

1.Header(头） 作用：记录令牌类型、签名算法等 例如：{“alg":"HS256","type","JWT}

2.Payload(有效载荷）作用：携带一些用户信息 有系统自带的部分信息，还可以追加自定义信息例如{"userId":"1","username":"mayikt"}

3.Signature(签名）作用：防止Token被篡改、确保安全性 例如 计算出来的签名，一个字符串。就是将header、payload的部分数据或者全部数据通过后端salt值和算法进行加密。前端返回token后也是拿header、payload的部分数据或者全部数据再算一遍和接收到的signature进行对比，相同则身份通过。

## 线程池

更详细解读见[还不懂Java线程池实现原理，看这一篇文章就够了 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/583191578)

虽然有多种创建线程池的方法，但是目前还是推荐new ThreadPoolExecutor，手动创建

```java
public class ThreadPoolDemo {

    public static void main(String[] args) {
        // 1. 创建线程池
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                3,
                3,
                0L,
                TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<>(),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());
      
        // 2. 往线程池中提交3个任务
        for (int i = 0; i < 3; i++) {
            final int index = i;  // final关键字使下面能打印出期望的i
            threadPoolExecutor.execute(() -> {
                System.out.println(index);
                System.out.println(Thread.currentThread().getName() + " 关注公众号：一灯架构");
            });
        }
      
        // 3. 关闭线程池
        threadPoolExecutor.shutdown();
    }
}
// 输出结果：
// pool-1-thread-2 关注公众号：一灯架构
// pool-1-thread-1 关注公众号：一灯架构
// pool-1-thread-3 关注公众号：一灯架构
```
### 参数

|参数名称|参数含义|
|:----|:----|
|int corePoolSize|核心线程数|
|int maximumPoolSize|最大线程数|
|long keepAliveTime|线程存活时间|
|TimeUnit unit|时间单位|
|BlockingQueue workQueue|阻塞队列|
|ThreadFactory threadFactory|线程创建工厂|
|RejectedExecutionHandler handler|拒绝策略|

1. corePoolSize 核心线程数
当往线程池中提交任务，会创建线程去处理任务，直到线程数达到corePoolSize，才会往阻塞队列中添加任务。默认情况下，空闲的核心线程并不会被回收，除非配置了allowCoreThreadTimeOut=true。

2. maximumPoolSize 最大线程数
当线程池中的线程数达到corePoolSize，阻塞队列又满了之后，才会继续创建线程，直到达到maximumPoolSize，另外空闲的非核心线程会被回收
3. keepAliveTime 线程存活时间
非核心线程的空闲时间达到了keepAliveTime，将会被回收。
4. 
TimeUnit 时间单位
线程存活时间的单位，默认是TimeUnit.MILLISECONDS（毫秒），可选择的有：
TimeUnit.NANOSECONDS（纳秒）
TimeUnit.MICROSECONDS（微秒）
TimeUnit.MILLISECONDS（毫秒）
TimeUnit.SECONDS（秒）
TimeUnit.MINUTES（分钟）
TimeUnit.HOURS（小时）
TimeUnit.DAYS（天）
5. workQueue 阻塞队列
当线程池中的线程数达到corePoolSize，再提交的任务就会放到阻塞队列的等待，默认使用的是LinkedBlockingQueue，可选择的有：
LinkedBlockingQueue（基于链表实现的阻塞队列）
ArrayBlockingQueue（基于数组实现的阻塞队列）
SynchronousQueue（只有一个元素的阻塞队列）
PriorityBlockingQueue（实现了优先级的阻塞队列）
DelayQueue（实现了延迟功能的阻塞队列）
6. threadFactory 线程创建工厂
用来创建线程的工厂，默认的是Executors.defaultThreadFactory()，可选择的还有Executors.privilegedThreadFactory()实现了线程优先级。当然也可以自定义线程创建工厂，创建线程的时候最好指定线程名称，便于排查问题。
7. RejectedExecutionHandler 拒绝策略
当线程池中的线程数达到maximumPoolSize，阻塞队列也满了之后，再往线程池中提交任务，就会触发执行拒绝策略，默认的是AbortPolicy（直接终止，抛出异常），可选择的有：
AbortPolicy（直接终止，抛出异常）
DiscardPolicy（默默丢弃，不抛出异常）
DiscardOldestPolicy（丢弃队列中最旧的任务，执行当前任务）
CallerRunsPolicy（返回给调用者执行）
### 传值

多个线程里面如果访问普通的数据类型会报错：多个线程间不能保证得到的是最新的数据。两种解决方法

1. final关键字
```java
for (int i = 0; i < 200; i++) {
    final int index = i;
    threadPoolExecutor.execute(() -> {
        System.out.println("================" + index);
    });
}
```
2. atomic(原子的)
```java
AtomicInteger counter = new AtomicInteger(0);
for (int i = 0; i < 10; i++) {
    int index = counter.incrementAndGet();
    threadPoolExecutor.execute(() -> {
        System.out.println("================index:" + index);
    });
}
```
### 线程数

由于cpu数量/核心数的限制通常我们可以取一个最小值是cpu核心数（大部分情况适用）。上限就比较难判断但是简略的原则是：

$$最小值：cpu核心数$$

$$最大值：线程等待时间(TWT)、核心数成正比(CPUC)。线程处理时间成反比(THT)。\frac{TWT}{THT}*CPUC$$

##  sql的批处理

```java
@Autowired
    private SqlSessionFactory sqlSessionFactory;
Date date = new Date();
            SimpleDateFormat dateFormat= new SimpleDateFormat("yyyy-MM-dd :hh:mm:ss");
            System.out.println("================" + dateFormat.format(date) + "start" + "================");
            SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH,false);
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            for (int i = 0; i < 1000000; i++) {
                String QRcodeRandom = IdUtil.simpleUUID();
                userMapper.temp(QRcodeRandom, new Date().toString());
            }
            sqlSession.commit();
            sqlSession.close();
            Date date1 = new Date();
            System.out.println("================" + dateFormat.format(date1) + "end" + "================");
```
节约了每次插入与数据库连接的时间，一次连接任务完成后一次提交，向三列其中两列唯一索引，一列主键索引的表中插入100w条数据耗时1小时20分钟。
但是如果开启了多线程，同时再开启批处理通过实验发现性能反而下降，启用4线程向上述表中插入1w条数据耗时15s，如果同时开启批处理耗时达到30s，为什么会这样还需要再明确

## Redis限流

```sql
@Configuration
public class RequestLimitConfig {

    @Autowired
    private RedisTemplate<String,Object> redisTemplate;

    @Bean
    public CustomKeyGenerator customKeyGenerator() {
        return new CustomKeyGenerator();
    }

    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(redisTemplate);
    }
}

public class CustomKeyGenerator implements KeyGenerator {
    @Override
    public Object generate(Object o, Method method, Object... objects) {
        // 派生出唯一键值代表的限制器对应输出流量控制符信息
        String key = o.getClass().getSimpleName() + "_" + method.getName() + "_" + Arrays.toString(objects);
        System.out.println("Create the cache key: " + key);
        return key;
    }
}

@Slf4j
public class RedisRateLimiter {

    private final RedisTemplate<String,Object> redisTemplate;

    private static final String REDIS_NAME = "RequestLimiting";

    // 总访问频率
    private static final int TOTAL_RATE = 10;

    // 针对全局用户请求频率的时间区间
    private static final long TOTAL_RATE_INTERVAL_SEC = 60;

    // 针对单个用户请求频率的时间区间
    private static final long USER_RATE_INTERVAL_SEC = 5;

    // 针对单个用户的总访问频率
    private static final int USER_TOTAL_RATE = 3;

    public RedisRateLimiter(RedisTemplate<String,Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public boolean rateLimit(HttpServletRequest request){
        String ip = getRemoteHost(request);
        String uri = request.getRequestURI();

        String totalKey = REDIS_NAME + "_Total_" + uri + "_" + System.currentTimeMillis() / (TOTAL_RATE_INTERVAL_SEC * 1000);
        String userKey = REDIS_NAME + "_User_" + ip + "_" + uri + "_" + System.currentTimeMillis() / (USER_RATE_INTERVAL_SEC * 1000);

        int totalCurrent = getCounterValue(totalKey);
        int userCurrent = getCounterValue(userKey);

        // 全局访问频率控制，针对全局用户的访问时间区间
        if (totalCurrent > TOTAL_RATE) {
            log.error("The global frequency limit has been exceeded.");
            return false;
        }

        // 针对单个用户的访问频率控制，增加针对当前用户的IP地址限流
        if (userCurrent > USER_TOTAL_RATE) {
            log.error("The personal frequency limit has been exceeded.");
            return false;
        }

        increment(totalKey);
        increment(userKey);
        return true;
    }

    private int getCounterValue(String key){
        ValueOperations<String,Object> operations = redisTemplate.opsForValue();
        if(!redisTemplate.hasKey(key)){
            operations.set(key,0, Duration.ofSeconds(TOTAL_RATE_INTERVAL_SEC));
        }
        Object counter = operations.get(key);
        return Integer.parseInt(counter.toString());
    }

    private void increment(String key){
        ValueOperations<String,Object> operations = redisTemplate.opsForValue();
        operations.increment(key,1);
    }

    private String getRemoteHost(HttpServletRequest request) {
        String ip = request.getHeader("X-Real-IP");
        if (!Strings.isNullOrEmpty(ip)) {
            if (ip.indexOf(",") > -1) {
                String[] ipsArr = ip.split(",");
                for (int i = 0; i < ipsArr.length; ++i) {
                    String oneIp = ipsArr[i];
                    if (!("unknown".equalsIgnoreCase(oneIp))) {
                        ip = oneIp;
                        break;
                    }
                }
            }
        } else {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}

// 在controller的某个请求方法上添加，
@GetMapping("/some-api")
public ResponseEntity<?> someApi(HttpServletRequest request){
        if(redisRateLimiter.rateLimit(request)){
            // 在此处处理业务逻辑
            ...
            return new ResponseEntity<>(HttpStatus.OK);
        }else{
            // 在此处做限制响应或者直接抛出非法访问异常
            throw new RuntimeException("Illegal request has been detected.");
        }
}
```
## 获取tif元数据

```xml
<dependencies>
  <dependency>
    <groupId>com.drewnoakes</groupId>
    <artifactId>metadata-extractor</artifactId>
    <version>2.15.0</version>
  </dependency>
</dependencies>
```

---
```java
import com.drew.imaging.ImageMetadataReader;
import com.drew.metadata.Directory;
import com.drew.metadata.Metadata;
import com.drew.metadata.Tag;

import java.io.File;

public class DemoApplication {

    public static void main(String[] args) {
        String tiffPath = "path/to/your/tiff/file.tif";
        File tiffFile = new File(tiffPath);

        try {
            Metadata metadata = ImageMetadataReader.readMetadata(tiffFile);
            
            // 遍历所有目录（Directory）
            for (Directory directory : metadata.getDirectories()) {
                // 遍历目录中的标签（Tag）
                for (Tag tag : directory.getTags()) {
                    System.out.println(tag);
                }
            }
          // 获取图像的宽度和高度
          int imageWidth = metadata.getFirstDirectoryOfType(ExifIFD0Directory.class).getInt(ExifIFD0Directory.TAG_IMAGE_WIDTH);
          int imageHeight = metadata.getFirstDirectoryOfType(ExifIFD0Directory.class).getInt(ExifIFD0Directory.TAG_IMAGE_HEIGHT);
          
          System.out.println("Image Width: " + imageWidth);
          System.out.println("Image Height: " + imageHeight);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
## springboot黑窗口与用户互动

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import java.util.Scanner;

@SpringBootApplication
public class MainApplication implements CommandLineRunner {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }

    @Override
    public void run(String... args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("请输入文件名: ");
        String fileName = scanner.nextLine();
        System.out.println("您输入的文件名是：" + fileName);

        System.out.print("请输入文件夹名: ");
        String folderName = scanner.nextLine();
        System.out.println("您输入的文件夹名是：" + folderName);

        // 根据用户输入的文件名和文件夹名进行后续处理
        // ...

        scanner.close();
    }
}
```
## 简单正则表达应用

```java
String pattern = "作业 (.*?) 打印完成！";
  Pattern regex = Pattern.compile(pattern);
  Matcher matcher = regex.matcher(line);
  if (matcher.find()) {
      String extractedContent = matcher.group(1);
      System.out.println(extractedContent);
  }
```
## 工厂模式、单例模式

* 单例模式
核心理念就是私有化构造函数，不允许从外部new myClass来实例化，来达到全局只有唯一实例的目的。

```java
public class Singleton {
    private static Singleton instance;
    private Singleton() {
        // 私有的构造函数
        System.out.println("具体sing的操作");
    }
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
// Singleton zzh = new Singleton();就不再被允许，虽然你可以将private改为public来实现外部实例化但是这也违背了单例模式的理念。
```
* 工厂模式
理念是通过工厂类来获得各种不同的实例，在扩展实例等操作的时候更加灵活，比如原本需要ConcreteProductA a = new ConcreteProductA();现在通过Factory f = new Factory();和Product a = f.getProduct("A");来获得，之后可能新增B、C。。。但是我觉得这个多此一举

```java
// 抽象产品类
public interface Product {
    void operation();
}
// 具体产品类A
public class ConcreteProductA implements Product {
    @Override
    public void operation() {
        System.out.println("具体产品A的操作");
    }
}
// 具体产品类B
public class ConcreteProductB implements Product {
    @Override
    public void operation() {
        System.out.println("具体产品B的操作");
    }
}
// 工厂类
public class Factory {
    public static Product getProduct(String type) {
        if (type.equalsIgnoreCase("A")) {
            return new ConcreteProductA();
        } else if (type.equalsIgnoreCase("B")) {
            return new ConcreteProductB();
        }
        return null;
    }
}
```

