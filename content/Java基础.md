### 1.	wait, sleep区别

1. `wait()`方法属于java.lang.Object, `sleep()`属于java.lang.Thread
2. `wait()`表示等待在一个对象上线,需要获得这个对象的锁,否则抛出`IllegalMonitorStateException`
3. 调用 `wait()`会自动释放获取到的锁,`sleep()`方法不会
4. 线程状态:`wait()`的线程状态为 WAITING,`sleep()` 为TIMED_WAITING



### 2.	wait, sleep 区别,调用 wait 方法后,如何唤醒线程

1. 区别略
2. 如何唤醒调用 `wait()`方法的线程
   1.  `wait()`的原理:这个方法需要获取对象的锁, 及`ObjectMonitor` 对象, 这个对象中维护了`_WaitSet` 和`_EntryList`, `wait()`方法就是把当前线程放入`_ WaitSet` 队列中, 再调用` park()`, 挂起当前线程. 最后释放这个锁
   2. 如何唤醒:就是在该对象上调用` notify()` 方法,该方法会在`_ WaitSet` 中随机选取一个线程 放入`_ EntryList` 中,就唤醒了这个线程



### 3.	volatile关键字怎么保证可见性

#### 3.1	明确什么是可见性

可见性是指对一个在堆上的变量的修改,每个线程都能看到这个变量的最新的值.

#### 3.2	为什么会出现可见性问题

java 因为跨平台,抽象了一套属于自己的内存模型.存在一个主内存和每个线程都拥有的本地内存.

程序是不能直接访问主内存的,对一个变量的修改,需要将其copy 到本地内存,再修改后刷新会主内存,才完成了对这个变量的修改.如果此时一个线程对变量进行了修改,另外一个线程是看不见这个修改的

#### 3.3 如何保证

volatile 修饰的变量,会建立一个 happen-before 关系.及对于一个 volatile 变量的写 hb 与一个 volatile 变量的写.

volatile 是通过插入内存屏障与禁止指令重排序来保证 volatile 变量的可见性的.

读操作之前插入 loadload 屏障,读操作之后插入 loadstore 屏障

写操作之前插入 storestore 屏障,写操作之后插入storeload 屏障

再底层原理,在x86 架构上,编译成的本地代码,对这些变量的操作,会插入一个 lock 指令.涉及到缓存一致性协议了~

