### 1. wait, sleep区别

1. `wait()`方法属于java.lang.Object, `sleep()`属于java.lang.Thread
2. `wait()`表示等待在一个对象上线,需要获得这个对象的锁,否则抛出`IllegalMonitorStateException`
3. 调用 `wait()`会自动释放获取到的锁,`sleep()`方法不会
4. 线程状态:`wait()`的线程状态为 WAITING,`sleep()` 为TIMED_WAITING



### 2. wait, sleep 区别,调用 wait 方法后,如何唤醒线程

1. 区别略
2. 如何唤醒调用 `wait()`方法的线程
   1.  `wait()`的原理:这个方法需要获取对象的锁, 及`ObjectMonitor` 对象, 这个对象中维护了`_WaitSet` 和`_EntryList`, `wait()`方法就是把当前线程放入`_ WaitSet` 队列中, 再调用` park()`, 挂起当前线程. 最后释放这个锁
   2. 如何唤醒:就是在该对象上调用` notify()` 方法,该方法会在`_ WaitSet` 中随机选取一个线程 放入`_ EntryList` 中,就唤醒了这个线程



### 3. volatile关键字怎么保证可见性

#### 3.1 明确什么是可见性

可见性是指对一个在堆上的变量的修改,每个线程都能看到这个变量的最新的值.

#### 3.2 为什么会出现可见性问题

java 因为跨平台,抽象了一套属于自己的内存模型.存在一个主内存和每个线程都拥有的本地内存.

程序是不能直接访问主内存的,对一个变量的修改,需要将其copy 到本地内存,再修改后刷新会主内存,才完成了对这个变量的修改.如果此时一个线程对变量进行了修改,另外一个线程是看不见这个修改的

#### 3.3 如何保证

volatile 修饰的变量,会建立一个 happen-before 关系.及对于一个 volatile 变量的写 hb 与一个 volatile 变量的写.

volatile 是通过插入内存屏障与禁止指令重排序来保证 volatile 变量的可见性的.

读操作之前插入 loadload 屏障,读操作之后插入 loadstore 屏障

写操作之前插入 storestore 屏障,写操作之后插入storeload 屏障

再底层原理,在x86 架构上,编译成的本地代码,对这些变量的操作,会插入一个 lock 指令.涉及到缓存一致性协议了~

### 4. 反射

#### 4.1 反射的作用

1. 可以于运行时加载,探知和使用编译期间完全未知的类
2. 程序在运行状态中, 可以动态加载一个只有名称的类, 对于任意一个已经加载的类,都能够知道这个类的所有属性和方法; 对于任意一个对象,都能调用他的任意一个方法和属性
3. 加载完类之后, 在堆内存中会产生一个Class类型的对象(一个类只有一个Class对象), 这个对象包含了完整的类的结构信息,而且这个Class对象就像一面镜子,透过这个镜子看到类的结构,所以被称之为:反射
4. 每个类被加载进入内存之后,系统就会为该类生成一个对应的java.lang.Class对象,通过该Class对象就可以访问到JVM中的这个类

#### 4.2 获取Class对象的方式
1. 对象的getClass()方法
2. 类的.class(最安全/性能最好)属性
3. 运用Class.forName(String className)动态加载类,className需要是类的全限定名(最常用)

#### 4.3 获取Class中的信息
- 构造器：`Constructor<T> getConstructor(Class<?>... parameterTypes)`
- 方法：`Method getMethod(String name, Class<?>... parameterTypes)`
- 属性：`Field getField(String name)`
- 注解：`<A extends Annotation A getAnnotation(Class<A> annotationClass)`
- 内部类：`Class<?>[] getDeclaredClasses()`
- 外部类：`Class<?> getDeclaringClass()`
- 所实现的接口：`Class<?>[] getInterfaces()`
- 修饰符：`int getModifiers()`

[https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html "Class使用文档")

#### 4.4 使用Class的反射
1. 创建对象
	1. 使用Class对象的newInstance()方法来创建该Class对象对应类的实例(这种方式要求该Class对象的对应类有默认构造器).
	2. 先使用Class对象获取指定的Constructor对象, 再调用Constructor对象的newInstance()方法来创建该Class对象对应类的实例(通过这种方式可以选择指定的构造器来创建实例).
	3. 场景使用：对象池技术
2. 调用方法
	1. 通过该Class对象的getMethod来获取一个Method数组或Method对象.每个Method对象对应一个方法,在获得Method对象之后,就可以通过调用invoke方法来调用该Method对象对应的方法
	2. 场景使用：属性注入
3. 访问成员变量
	1. 通过Class对象的的getField()方法可以获取该类所包含的全部或指定的成员变量Field,Filed提供了如下两组方法来读取和设置成员变量值
		1. getXxx(Object obj): 获取obj对象的该成员变量的值, 此处的Xxx对应8中基本类型,如果该成员变量的类型是引用类型, 则取消get后面的Xxx;
		2. setXxx(Object obj, Xxx val): 将obj对象的该成员变量值设置成val值.此处的Xxx对应8种基本类型, 如果该成员类型是引用类型, 则取消set后面的Xxx; 
	2. getDeclaredXxx方法可以获取所有的成员变量,无论private/public;
4. 使用反射获取泛型信息
	- ParameterizedType java.util.Set<java.lang.String>
	```
    Field field = Client.class.getDeclaredField("objectMap");
    Type gType = field.getGenericType();
	```
5. 使用反射获取注解信息
	
    
参考文档：[http://www.importnew.com/17616.html](http://www.importnew.com/17616.html "java反射")

### 5. long类型的操作是原子性吗？为什么？

要分情况讨论，对于64位的系统来说是的，对于32位的系统来说不是，为什么呢？

因为对于32位虚拟机来说，每次原子读写是32位的，而long和double则是64位的存储单元，这样会导致一个线程在写时，操作完前32位的原子操作后，轮到B线程读取时，恰好只读取到了后32位的数据，这样可能会读取到一个既非原值又不是线程修改值的变量，它可能是“半个变量”的数值，即64位数据被两个线程分成了两次读取。



### 6. String 为什么 final 修饰

> 也就是问String 这个类为什么要设计成不可变的

1.字符串常量池

- jvm 存在字符串常量池,创建一个字符串,如果常量池中已经存在,会直接返回这个字符串的引用.如果能随意修改String 的值(修改常量池中的字符串),会出现问题.
- 其实大多数情况下相同的字符串变量他们指向的内存地址是相同的，大量使用字符串的情况下，可以节省内存空间，提高效率。但之所以能实现这个特性，String的不可变性是最基本的一个必要条件。要是内存里字符串内容能改来改去，这么做就完全没有意义了。

2.安全性

java 中大量使用String 字符串作为一个参数,比如` Socket socket = new Socket(host, port)`,如果 String 可修改,我们 会认为连接到某台机器,但实际上并没有

3.不可变对象带来的线程安全性

不可变对象天生就是线程安全的,避免锁操作,带来性能上的优势



### 7. java 为什么不支持多继承

多继承会带来菱形问题.存在一个抽象类 A, 有方法` foo()`, B,C 继承 A并且实现了各自的` foo()` 方法.如果允许多继承, D 继承 B,C 两个类,那么此时D 的` foo()` 存在歧义

### 8. 重构

**为什么要重构**

延续软件生命、适应需求变更、加深理解代码、提高自我编程能力

**何时要重构**

- 添加功能时重构
- 修补错误时重构
- 复审代码时重构

**何时不要重构**

- 无法稳定运行直接重写不用重构
- 项目已经接近最后期限，但是你没有足够的时间

**难点**

- 数据库(database)：结构紧密耦合在一起、数据迁移。
- 兼容新旧接口(interface)

**重构方法**

- 重新组织函数
- 在对象之间搬移特性
- 重新组织数据
- 简化函数调用
- 处理概括关系

