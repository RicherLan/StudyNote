# 杂知识
* 一些项目技术:https://mp.weixin.qq.com/s/_Ox67FpVi1P032EWyRlZXA
* 代码整洁之道书：https://www.bookstack.cn/read/Clean-Architecture-zh/docs-ch1.md
* ios memory deep dive


# 设计架构：
* clean architecture:https://www.bookstack.cn/read/Clean-Architecture-zh/docs-ch1.md
* 小明的mvvm：https://docs.corp.kuaishou.com/k/home/VM1xCx-qw9pA/fcAAOkWDwpwUcfS95pLdkZDq8

# android技术知识：
* view点击事件分发：https://www.jianshu.com/p/38015afcdb58
* window：https://juejin.cn/post/6888688477714841608
* dpi: https://www.jianshu.com/p/d8e6bb5deea5
* Context：https://mp.weixin.qq.com/s/AtaHv3tKjJniSRhavLx-jA
* 一张图片的大小:https://www.cnblogs.com/dasusu/p/9789389.html
* 高效加载大位图:https://developer.android.google.cn/topic/performance/graphics/load-bitmap?hl=zh-tw
* FlexboxLayout:https://juejin.cn/post/6844903697500241928#heading-19
* drawable:https://blog.csdn.net/guolin_blog/article/details/50727753
*  .9图：https://www.cnblogs.com/Free-Thinker/p/13182801.html


# Java技术知识点：
* happens-before:https://segmentfault.com/a/1190000011458941
* HashMap：https://tech.meituan.com/2016/06/24/java-hashmap.html
* synchronized: https://xiaomi-info.github.io/2020/03/24/synchronized/
* AQS(不完整):https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html
* 关闭线程池:http://www.justdojava.com/2019/09/10/threadpool-shutdown/

# kotlin


# 大前端
* js教学：https://zh.javascript.info/js?map

# server
* kafka:https://www.orchome.com/5

# 一些三方库
* Fresco：https://www.rousetime.com/2017/12/28/Fresco%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B9%8BProducer/
* Retrofit2: https://juejin.cn/post/6844903570605621255#heading-16
* Rxjava解析：https://docs.corp.kuaishou.com/d/home/fcACwhaVHTNZ_AbfStxFignJN
* dagger2:https://www.jianshu.com/p/6a65b9d33fea


# 操作系统
* 计算机编码（unicode ASCII）：https://www.cnblogs.com/gavin-num1/p/5170247.html
* Cache:https://zhuanlan.zhihu.com/p/102293437


# 算法
* LRU：https://leetcode.com/problems/lru-cache/description/
* LFU：https://leetcode.com/problems/lfu-cache/description/


# 一些好用的工具
*tmux：http://louiszhai.github.io/2017/09/30/tmux/

TODO： 
* mmkv


### 算法：
LRU：如何用LinkedHashMap实现 https://juejin.cn/post/6844903917524893709, LinkedHashMap天然就是LRU


### gc
gc系列文章 https://www.zhihu.com/column/c_1293612595426095104


### IO
现在有n个IO事件，我们如何同时处理多个流
1. 非堵塞忙轮询：如果所有的流都没有数据，那么只会白白浪费cpu
    while(true) {
        for(i : stream[]) {
            if(i has data) {
                read data until unavailable
            }
        }
    }

2. 堵塞：从select可以知道，有IO事件发生了，但不知道是那哪个流(1个、多个、全部)，只能无差别轮训所有的流，找出可以读或者可以写入的流，进行操作
    while(true) {
        select(stream[]) // 堵塞
        for(i : stream[]) {
            if(i has data) {
                read data until unavailable
            }
        }
    }

3. linux 2.3之后，epoll, 会把哪个流发生了怎样的io事件通知我们，此时我们对这些流的操作都是有意义的，复杂度降低到O(1)
     while(true) {
        active_stream[] = epoll_wait()
        for(i in active_stream[]) {
            read or write until unavailable
        }
    }
epoll是Linux中最高效的IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作
epoll有3个方法，epoll_create()， epoll_ctl()，epoll_wait()_
** epoll_create :创建epoll句柄**
    int epoll_create(int size)；
    ：用于创建一个epoll的句柄，size是指监听的描述符个数， 现在内核支持动态扩展，该值的意义仅仅是初次分配的fd个数，后面空间不够时会动态扩容。 当创建完epoll句柄后，占用一个fd值.

    使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

** epoll_ctl：执行对需要监听的文件描述符(fd)的操作**
    int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；

    用于执行对需要监听的文件描述符(fd)的操作,比如EPOLL_CTL_ADD
    fd：需要监听的文件描述符；
    epoll_event：需要监听的事件

** epoll_wait：等待事件的到来**
    int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);

    等待事件的到来，有三种情况会触发返回
    1. 发生错误 返回值为负数
    2. 等待超时（timeout）
    3. 监测到某些文件句柄上有事件发生

** epoll和eventfd结合使用**
    eventfd中无数据，线程触发epoll_wait()的等待，该线程处于阻塞，另外一个线程通过往eventfd中write数据，使描述符可读，epoll返回，这就达到了唤醒的目的。_


epoll、eventId相关
eventfd 原理与实践：https://juejin.cn/post/7146239048191836190
eventfd与epoll的使用：https://blog.csdn.net/SCHOLAR_II/article/details/127361070
native——handler: https://juejin.cn/post/7146239048191836190

pipe的4次拷贝：https://blog.csdn.net/weixin_39965673/article/details/112073414
android 6的handler使用epoll+pipe的方式实现：https://devbins.github.io/post/pipe/


## mmap
[不错的文章](https://www.cnblogs.com/huxiao-tee/p/4660352.html)
总而言之，常规文件操作需要从磁盘到页缓存再到用户主存的两次数据拷贝。而mmap操控文件，只需要从磁盘到用户主存的一次数据拷贝过程。
说白了，mmap的关键点是实现了用户空间和内核空间的数据直接交互而省去了空间不同数据不通的繁琐过程。因此mmap效率更高。

## 线程、锁
java线程：
suspend和stop弃用原因：
1.suspend()，在调用该方法暂停线程的时候，线程由running状态变成blocked，需要等待resume方法将其重新变成runnable。
而线程由running状态变成blocked时，只释放了CPU资源，没有释放锁资源，可能出现死锁。
比如：线程A拿着锁1被suspend了进入了blocked状态，等待线程B调用resume将线程A重新runnable。但是线程B一直在lock pool中等待锁1,线程B要拿到锁1才能running去执行resumeA。这就死锁了。

2.stop()，调用stop方法无论run()中的逻辑是否执行完，都会释放CPU资源，释放锁资源。这会导致线程不安全。
比如：线程A的逻辑是转账（获得锁，1号账户减少100元，2号账户增加100元，释放锁），那线程A刚执行到1号账户减少100元就被调用了stop方法，释放了锁资源，释放了CPU资源。1号账户平白无故少了100元。一场撕逼大战开始了。


Thread.interrupt方法，在线程wait、sleep等时，会抛出interruptedException异常 且 会把中断标记位重新置为false(方案：可以在catch内在调用intercept())
处于死锁状态时，不会理会中断


一写多读的场景适合用volatile

// notifyAll()写在这两个地方没有区别，notifyAll会通知wait的那些线程进就绪，等notifyAll所在的synchronized块执行完了后，wait的那些线程才会去执行
public void fun() {
    synchronized(this) {
        ....
       // notifyAll()；
        ....
        notifyAll()；
    }
}
public void fun2() {
    synchronized(this) {
        while(xxx) {
            wait()
        }
        ...
    }
}


管程
指的是管理共享变量以及共享变量的操作过程，让它们支持并发。

synchronized：https://juejin.cn/post/6888137809929093133
偏向锁为啥废弃，谈谈Java Synchronized 的锁机制，以及管程：https://cloud.tencent.com/developer/article/1759559
轻量级锁、锁升级：https://bbs.huaweicloud.com/blogs/335820

锁升级的图：https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/feec7b70b0f944c4aa19899d5d0c7db4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp

锁优化
Synchronized 只在 JDK 1.6 以前性能才很差，因为这之前的 JVM 实现都是重量级锁，直接调用 ObjectMonitor 的 enter 和 exit。从 JDK 1.6 开始，HotSpot 虚拟机就增加了上述所说的几种优化：

偏向锁
轻量级锁
自旋锁
其余还有：

适应性自旋
锁消除
锁粗化

锁消除
这属于编译器对锁的优化，JIT 编译器在动态编译同步块时，会使用逃逸分析技术，判断同步块的锁对象是否只能被一个对象访问，没有发布到其它线程。
如果确认没有“逃逸”，JIT 编译器就不会生成 Synchronized 对应的锁申请和释放的机器码，就消除了锁的使用。

锁粗化
JIT 编译器动态编译时，如果发现几个相邻的同步块使用的是同一个锁实例，那么 JIT 编译器将会把这几个同步块合并为一个大的同步块，从而避免一个线程“反复申请、释放同一个锁“所带来的性能开销。

减小锁粒度
我们在代码实现时，尽量减少锁粒度，也能够优化锁竞争。

总结
其实现在 Synchronized 的性能并不差，偏向锁、轻量级锁并不会从用户态到内核态的切换；只有在竞争十分激烈的时候，才会升级到重量级锁。
Synchronized 的锁是由 JVM 实现的。
偏向锁已经被废弃了。

aqs
https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html
https://xie.infoq.cn/article/0223d5e5f19726b36b084b10d

比如线程加入等待队列的时候，需要把tail尾节点设置成当前节点，设置方法：
class AbstractQueuedSynchronizer {
    private transient volatile Node tail;
    static {
        tailOffset = unsafe.objectFieldOffset (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
    }
    private final boolean compareAndSetTail(Node expect, Node update) {
        // cas设置对象中某field的时候， 替换该field地址上的数据(新node对象的地址)
        // 如果tailOffset的Node和Expect的Node地址是相同的，那么设置Tail的值为Update的值
        return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
    }
}

java.util.concurrent中对aqs的应用场景：
ReentrantLock	        使用AQS保存锁重复持有的次数。当一个线程获取锁时，ReentrantLock记录当前获得锁的线程标识，用于检测是否重复获取，以及错误线程试图解锁操作时异常情况的处理。
Semaphore	            使用AQS同步状态来保存信号量的当前计数。tryRelease会增加计数，acquireShared会减少计数。
CountDownLatch    	    使用AQS同步状态来表示计数。计数为0时，所有的Acquire操作（CountDownLatch的await方法）才可以通过。
ReentrantReadWriteLock	使用AQS同步状态中的16位保存写锁持有的次数，剩下的16位用于保存读锁的持有次数。
ThreadPoolExecutor    	Worker利用AQS同步状态实现对独占线程变量的设置（tryAcquire和tryRelease）。


cpu密集型，线程池的线程数量一般是：cpu核数+1
加1：即使当计算密集型的线程偶尔由于缺页或者其他原因而暂停时，这个额外的线程也能确保cpu的时钟周期不会被浪费


## finilize和finally
Object的finilize方法不一定执行
守护线程的finally块不一定执行

finalize方法
1.对象未覆盖 finalize 方法，不执行
2.finalize已经被调用过一次，不执行：因此在finalize方法中把this重新被其他对象引用一下就可以防止死亡了，但是后面也不会在被调用finalize方法了
3.jvm不保证finalize能完整执行
以上是不一定会被调用，至于确定需要调用的时候会放入f-queue , f-queue 属于低优先级 Finalize 线程，不知道什么时候执行队列，所以不确定什么时候调用。



## ThreadLocal
不存数据，类似于一个工具方法，真正存放数据的是Thread.threadLocalMap
 public T get() {
    Thread t = Thread.currentThread();
    // getMap(t)：return t.threadLocals;
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
// 每个线程内部都有ThreadLocalMap成员
static class ThreadLocalMap {
    /**
        * The entries in this hash map extend WeakReference, using
        * its main ref field as the key (which is always a
        * ThreadLocal object).  Note that null keys (i.e. entry.get()
        * == null) mean that the key is no longer referenced, so the
        * entry can be expunged from table.  Such entries are referred to
        * as "stale entries" in the code that follows.
        */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    
    private static final int INITIAL_CAPACITY = 16;

    /**
        * The table, resized as necessary.
        * table.length MUST always be a power of two.
        */
    private Entry[] table;

    // ThreadLocal.get方法最终会调用到当前线程的threadLocalMap的这个方法
    private Entry getEntry(ThreadLocal<?> key) {

    }
}

内存泄漏
ThreadLocal的原理和内存泄露分析：https://juejin.cn/post/7091649401629704223
内存泄漏源码分析https://www.jianshu.com/p/dde92ec37bd1

ThreadLocalMap 中的 key 使用了弱引用，会导致 value 出现内存泄漏。
case： 假设在业务代码中使用完 ThreadLocal，threadLocalRef 被回收了
    由于 ThreadLocalMap 只持有 ThreadLocal 的弱引用，没有任何强引用指向 threadlocal 实例，所以 threadlocal 就可以顺利被 GC 回收，此时 Entry 中的 key = null
    在没有手动删除这个 Entry 以及 CurrentThread 依然运行的前提下，也存在有强引用链 threadRef -> currentThread -> threadLocalMap -> entry -> value，value 不会被回收，而这块 value 永远不会被访问到了，导致 value 内存泄漏
导致内存泄漏的原因
    1. 没有手动删除相应的 Entry 对象
    2. 当前线程依然在运行
如何解决内存泄露
    1. 使用完 ThreadLocal，调用其 remove 方法删除对应的 Entry
        class ThreadLocal {
             public void remove() {
                ThreadLocalMap m = getMap(Thread.currentThread());
                if (m != null)
                    m.remove(this);
            }
        }
        class ThreadLocalMAp {
            private void remove(ThreadLocal<?> key) {
                Entry[] tab = table;
                int len = tab.length;
                int i = key.threadLocalHashCode & (len-1);
                for (Entry e = tab[i];
                    e != null;
                    e = tab[i = nextIndex(i, len)]) {
                    if (e.get() == key) {
                        e.clear(); // weakReference.clearReferent(), 此时只是回收key而已
                        expungeStaleEntry(i); // 这里给entry置空的
                        return;
                    }
                }
            }
        }
    2. 使用完 ThreadLocal，当前 Thread 也随之运行结束（不好控制，线程池中的核心线程不会销毁）

延伸：
threadLocalMap为啥不用weakHashMap呢，而是自己写了一个类:ThreadLocalMap；weakHashMap的entry也是弱引用，而且比threadLocal好的一点是，他对value也会释放
threadLocal释放value是有问题的，如果使用不当，很容易内存泄漏
    1. 线程退出(Thread.exit())中会让threadLocalMap=null：这里需要注意线程池的复用不会停止线程
    2. threadLocal.remove
    3. threadLocal的get和set(只有hash冲突的时候发现key为null了才会去让value=null)

我觉得的一个点是，threadLocalMap的hash方式是开放地址法，可以节省空间，但感觉这并不是主要原因



## Android启动流程
<img width="842" alt="image" src="https://user-images.githubusercontent.com/49143666/231740244-9ec3b3b2-dea5-4411-b201-c92455319159.png">
<img width="499" alt="image" src="https://user-images.githubusercontent.com/49143666/234188880-92a6d8e7-186a-4d94-81b5-259029f765e7.png">
<img width="531" alt="image" src="https://user-images.githubusercontent.com/49143666/234189022-35eca2e8-177d-4d2e-9da8-bbc46a4de493.png">
一些关键性的进程如果杀死了，系统会重启，比如init进程、zygoyte进程等

* 进程、jvm、native、内核空间、用户空间的关系：
<img width="854" alt="image" src="https://user-images.githubusercontent.com/49143666/234202332-4ed5c32c-e294-4f89-aa3e-2dbf32d05659.png">


##### init进程
<img width="980" alt="image" src="https://user-images.githubusercontent.com/49143666/234253571-6a52617a-8fe2-4f7e-988e-e52627fff699.png">

##### init创建的进程有哪些呢
如Zygite进程、Audio进程、SurfaceFlinger进程等都是由init进程创建的
<img width="1016" alt="image" src="https://user-images.githubusercontent.com/49143666/234254385-edfb6340-c83e-423d-8d23-593ca570e342.png">

###### init处理的重要事情
1. 挂载文件
2. 设置selinux(安全策略)
3. 开启属性服务，设置到epoll中
4. 解析init.rc
5. 循环执行脚本(init.rc解析出来的) --  如init.rc文件中有import zygote.rc, 然后有启动zygote的相关语句
<img width="810" alt="image" src="https://user-images.githubusercontent.com/49143666/234188176-93cb33cd-2612-4c93-bafa-579fba99bd37.png">
6. 循环等待

##### zygote进程
##### zygote工作内容
<img width="1023" alt="image" src="https://user-images.githubusercontent.com/49143666/234255397-fec29005-1c3f-4396-9560-fd6185477caa.png">

init_zygote32.rc、init_zygote64.rc、init_zygote64_32.rc等，注意文件中的启动参数
<img width="1041" alt="image" src="https://user-images.githubusercontent.com/49143666/234189524-b122e9e3-26a1-453e-bcc3-8f5ad2be1ab0.png">
app_process/Android.bp文件中声明了app_main.cpp, 该文件中的main方法来启动zygote和SystemServer
<img width="911" alt="image" src="https://user-images.githubusercontent.com/49143666/234194710-7f35b231-59fb-4723-8e0c-0bcfd7ed5e19.png">
<img width="860" alt="image" src="https://user-images.githubusercontent.com/49143666/234195447-0445228b-3bcb-435e-afb2-d6056b2314c8.png">
<img width="844" alt="image" src="https://user-images.githubusercontent.com/49143666/234193797-efc3e16e-559d-4b43-8955-34a0bf0f7d59.png">
<img width="873" alt="image" src="https://user-images.githubusercontent.com/49143666/234194533-24402cce-b4c3-480b-a963-390f47c5ace4.png">

###### zygote的native的启动细节：
<img width="892" alt="image" src="https://user-images.githubusercontent.com/49143666/234208102-8489dcfd-46e5-4716-b24c-58a00fe1cbdd.png">
最终调用到ZygoteInit.java#main

###### zygote的java启动细节：ZygoteInit.java
1. preload(xxx) // 预加载，加快进程的启动
<img width="755" alt="image" src="https://user-images.githubusercontent.com/49143666/234228775-b13d57fd-05aa-4803-9848-55752f7c3c62.png">
2. zygoteServer = new ZygoteServer(isPrimaryZygote) // 里面是socket，比如ams请求创建进程就是socket通信zygote的
3. Runnable r = forkSystemServer // 启动SystemServer进程，其中AMS、WMS、PMS等九十多个服务都在SystemServer进程
4. caller = zygoteServer.runSelectLoop(abiList) // 死循环，接收AMS传过来的消息

###### zygote总结
<img width="519" alt="image" src="https://user-images.githubusercontent.com/49143666/234242053-b0b1d2ef-fae8-47c4-b045-e6b81d5fc818.png">

#### SystemServer进程
##### zygote的forkSystemServer分析

##### SystemServer进程的启动流程分析

##### SystemServer进程的执行
<img width="997" alt="image" src="https://user-images.githubusercontent.com/49143666/234517113-2ab335ae-e2ea-4cb9-8bcf-e0b465be4f40.png">
<img width="683" alt="image" src="https://user-images.githubusercontent.com/49143666/234520875-e17354c3-5ea7-4a06-8512-ecc2cdf12880.png">
<img width="935" alt="image" src="https://user-images.githubusercontent.com/49143666/234521453-b02e53e3-bba1-4749-92a7-d778ec348108.png">
SystemServer.main中执行createSystemContext,该函数内会执行ActivityThread.systemMain(),该函数会new ActivityThread()，然后rhread.attach(true, 0) 注意这个true，ActivityThread也会有自己的context和application
<img width="743" alt="image" src="https://user-images.githubusercontent.com/49143666/234518935-c12affab-9e56-4474-8e90-27ee7320fc9f.png">

##### SystemServer如何管理服务
<img width="854" alt="image" src="https://user-images.githubusercontent.com/49143666/234543340-ae0f4f25-917f-42b8-b23e-e7a935010f62.png">
<img width="814" alt="image" src="https://user-images.githubusercontent.com/49143666/234543482-c9963d93-45ac-42bc-945e-7483d83b8efc.png">


##### SystemServer、SystemServiceManager、SystemService、ServiceManager
<img width="967" alt="image" src="https://user-images.githubusercontent.com/49143666/236172015-f1e60cf4-eb29-46d3-8f59-b452b0164543.png">
* SystemServer：进程，管理service
* SystemServiceManager具体管理service的东西
* SystemService：所有的服务继承自SystemService
    ActivityTaskManagerService extends IActivityTaskManagerService.Stub，ActivityTaskManagerService有一个静态内部类Lifecycle extends SystemService，这个内部类中有一个ActivityTaskManagerService mService成员。由此可见，解决java多继承的方法之一可以这样。
<img width="690" alt="image" src="https://user-images.githubusercontent.com/49143666/234547022-41fc7b06-d81d-4fa0-b4c3-c6007f59c0df.png">
* ServiceManager：进程
atms注册到ServiceManager中
<img width="792" alt="image" src="https://user-images.githubusercontent.com/49143666/236171121-0ef1ea6e-bd73-472f-b51d-e7b762bfc384.png">
<img width="762" alt="image" src="https://user-images.githubusercontent.com/49143666/236171369-6c728355-0a8f-49b2-a8cf-c98fc648557f.png">



## AMS
#### 启动：SystemServer中启动(startBootService函数中)
AMS：
<img width="763" alt="image" src="https://user-images.githubusercontent.com/49143666/236179794-f96f67ac-5f0b-4f5d-8bd2-49324120b8e8.png">
ATMS：
<img width="686" alt="image" src="https://user-images.githubusercontent.com/49143666/236173946-7acf07fe-12a3-432e-a213-4f9bd1f4a78d.png">
LocalServices中添加服务：
<img width="735" alt="image" src="https://user-images.githubusercontent.com/49143666/236179383-6d701bec-92fa-44d8-8b52-3792aff2f03e.png">

* AMS做的事情很多
在SystemServer中创建好AMS后，会调用AMS的setSystemProcess方法，该方法里创建了很多其他服务(如电源管理、内存服务、权限、进程信息等)并添加到了ServiceManager：
<img width="1155" alt="image" src="https://user-images.githubusercontent.com/49143666/236180850-a0b53f9e-9087-488a-ab0f-81e890541e60.png">

#### Application应用启动流程
##### 整体图：
<img width="581" alt="image" src="https://user-images.githubusercontent.com/49143666/236182258-1440749d-0e18-4fd0-bbfa-97970be4d0e8.png">
##### ActivityStackSuperVisor启动Activity的时候，判断如果进程是否存在
<img width="867" alt="image" src="https://user-images.githubusercontent.com/49143666/236191521-e4451eab-ac5f-4eaa-95b4-2f94b401c744.png">
##### 应用进程的创建启动
<img width="1034" alt="image" src="https://user-images.githubusercontent.com/49143666/236396306-dfca7007-83d5-459a-a03d-9a4d5d74db21.png">
* ProcessRecord：描述应用进程的信息，注意看里面的数据：如ApplicationInf,IApplicationThread,Pid等，很重要
<img width="805" alt="image" src="https://user-images.githubusercontent.com/49143666/236383241-4b2efe25-b9f1-40d0-9c06-2304409de5f8.png">
* ProcessList
是AMS中的属性成员，ProcessList中有一个List<ProcessRecord> mLruProcesses表示正在运行的application，排序方式是最近使用。
进程优先级的枚举也是在这个类里面定义的
* 创建应用进程分析
AMS.startProcessLocked(processName,APplicationInfo, ...) --> ProcessList.startProcessLocked(processName,APplicationInfo, ...) --> ProcessList.startProcess() --> Process.start --> ZYGOTE_PROCESS.start --> ZygoteProcess.zygoteSendAndGetResult --> ZygoteServer#runSelectLoop函数在一直监听消息(该方法是在ZygoteInit#main方法中调用的) --> ZygoteConnection.processOneCommand(ZygoteServer) --> Zygote.forkAndSpecialize --> nativeForkAndSpecialize
<img width="1137" alt="image" src="https://user-images.githubusercontent.com/49143666/236384045-6732fdb4-0055-4971-ba5f-1972626d2f28.png">
<img width="1145" alt="image" src="https://user-images.githubusercontent.com/49143666/236384409-146c03c4-6802-4fcb-99c1-bd097242fa1e.png">
<img width="1128" alt="image" src="https://user-images.githubusercontent.com/49143666/236384529-c4682c73-8761-4ff1-bca0-b7a4d33017ed.png">
<img width="1128" alt="image" src="https://user-images.githubusercontent.com/49143666/236384529-c4682c73-8761-4ff1-bca0-b7a4d33017ed.png">
<img width="1165" alt="image" src="https://user-images.githubusercontent.com/49143666/236384881-48d9f98b-aaa3-47e1-ba65-3c0a73ab3d27.png">
<img width="1192" alt="image" src="https://user-images.githubusercontent.com/49143666/236385189-6d6c95b5-4f08-4d74-990d-c509127e8cde.png">
<img width="874" alt="image" src="https://user-images.githubusercontent.com/49143666/236385343-1b3ff7d1-16b6-4146-bdb0-fcdc3b981c20.png">
* zygote fork进程后返回分析
zygoteConnection.handleChildProc --> ZygoteInit.zygoteInit -->ZygoteInit.nativeZygiteInit( App_main.onZygoteInit() --> ProcessState::self() ) --> RuntimeInit.applicationInit --> 最终执行ActivityThread.main
<img width="1152" alt="image" src="https://user-images.githubusercontent.com/49143666/236388214-a6c56d7a-7c93-41e7-9fcf-2dc2de22f8e8.png">
<img width="1141" alt="image" src="https://user-images.githubusercontent.com/49143666/236389176-bac393b5-f5ba-4e1b-a952-00d91be21307.png">
<img width="1138" alt="image" src="https://user-images.githubusercontent.com/49143666/236389097-090ec298-4e03-489f-90f6-4210dd95e51d.png">
<img width="785" alt="image" src="https://user-images.githubusercontent.com/49143666/236389415-bc66ca7e-2e50-4873-b896-b8b7af3edcb0.png">
<img width="770" alt="image" src="https://user-images.githubusercontent.com/49143666/236389574-8c7f93d2-09c6-4c20-a0ae-a456085771d3.png">
<img width="952" alt="image" src="https://user-images.githubusercontent.com/49143666/236389739-ce4e8a19-df38-45c3-938d-27f27bc8b824.png">
<img width="1149" alt="image" src="https://user-images.githubusercontent.com/49143666/236389953-e5c28923-f9a4-468a-9843-8f4744d5613f.png">
RuntimeInit.applicationInit
<img width="1141" alt="image" src="https://user-images.githubusercontent.com/49143666/236391205-9974b59d-48b1-4b18-8ff1-b5172ba750cd.png">
<img width="770" alt="image" src="https://user-images.githubusercontent.com/49143666/236391059-e4c0cc5f-ab7e-4588-9d2d-e45f2fe5a2e5.png">
<img width="1171" alt="image" src="https://user-images.githubusercontent.com/49143666/236391418-6d8c583c-f9ea-4d55-adaf-26d769da6d93.png">
ActivityThread.main的执行, main中调用activityThread.attach 
<img width="1156" alt="image" src="https://user-images.githubusercontent.com/49143666/236391912-96c8ed0a-3bc6-44e2-872a-e84776bb42fc.png">
attach中调用Ams.attchApplication(ApplicationThread mAppThread, startseq)，传递mAppThread binder给ams
<img width="1106" alt="image" src="https://user-images.githubusercontent.com/49143666/236397323-2b9a7161-3b34-40cf-a8fc-397eadb5daa8.png">
<img width="1110" alt="image" src="https://user-images.githubusercontent.com/49143666/236398334-c986e39f-ad48-4ede-9d8f-adf29495ab12.png">




## handler
java层和native层的文章：https://juejin.cn/post/6973142800808280071

堵塞和唤醒的时机：https://blog.csdn.net/ajsliu1233/article/details/124629486

2种构造方法
1. public Handler(@NonNull Looper looper)
   public Handler(@NonNull Looper looper, @Nullable Callback callback)
   public Handler(@NonNull Looper looper, @Nullable Callback callback, boolean async)
2. public Handler(boolean async)、public Handler(@Nullable Callback callback, boolean async)
    {   
        // 如果不传looper的话，那么构造函数里会mLooper = Looper.myLooper()，如果为空那么抛异常，因此线程里面要使用handler的话，要在创建handler前先Looper.prepare()
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

内存泄漏：
1. handler一般是匿名内部类(持有外部类引用)，然后Message持有handler(msg.target = handler), 然后mq持有了msg，looper持有了mq，looper被thread.threadLocalMap.table[i].value引用
2. handle.post(Runnable) ，runnable也是匿名内部类(持有外部类引用)，被封装成msg，然后mq持有了msg，looper持有了mq，
    sThreadLocal.set(new Looper)中把looper存到了ThreadLocalMap.table中，thread.threadLocalMap.table(持有了looper,Entry是extends WeakReference<ThreadLocal<?>>，但是entry的value(looper)还是强引用
3. Looper中static Looper sMainLooper; // 因此Looper.sMainLooper也是gcroot
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
loop函数中msg处理完后会调用msg.recycleUnchecked()，这里target、callback等全部置空

handler的postDelay,最后执行的是sendMessageAtTime, time是SystemClock.uptimeMillis() + delayMillis,为啥不直接传递delayTime，而是传递一个时间呢
1. 排序
2. 有底层调用，下面的需要delay时间的话，会用这个提前算出来的减去当前时间

Message
采用享元设计模式(缓存复用)
class Message {
    // 锁
    public static final Object sPoolSync = new Object();
    // 池子，采用链表的形式，当有message回收时，采用头插法(sPool = recycledMsg)
    private static Message sPool;
    private static int sPoolSize = 0;
    // 复用池大小是50
    private static final int MAX_POOL_SIZE = 50;
}
为啥不new，而是用obtain从pool中获取呢？
1. 60hz时，16.7ms就会发送3个消息：msg、内存屏障、移除内存屏障；
   频繁的new message， 如果没有消息复用机制的话内存抖动会明显，且STW(Stop the world，为了保证gc的执行会短暂暂停业务线程)
2. 频繁的new message, 很容易出现很多的内存碎片，容易出现oom

Looper：
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
// prepare中创建了Looper对象并添加到sThreadLocal
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
// 构造函数中创建了mq和绑定了当前的线程
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
}

MessageQueue
多线程间共享
不同的线程通过handler塞消息，那是如何保证线程安全的？
    enqueueMessage(msg, timeMs)，内部都是synchronized (this)的
    next()函数也是synchronized (this)，因此取消息也涉及到链表的删除(尽管next只在looper运行所在的线程执行)
cpp层代码：https://cs.android.com/android/platform/superproject/+/master:frameworks/base/core/jni/android_os_MessageQueue.cpp;l=188;drc=master


native层hanler文章：https://juejin.cn/post/7146239048191836190
1. 堵塞唤醒机制为啥不用wait和notify()
 回答：wait和notify是java层的机制，c/c++层的开发者也需要handler这一套机制
2. 为啥使用的是epoll + eventfd，而不是pipe管道呢
 回答：android2.3的时候是使用epoll + pipe的，后面才改成了epoll + eventFd的。
    Eventfd在信号通知的场景下，相对比pipe有非常大的资源和性能优势， 本质其实是counter(计数器)和channel(数据信道)的区别。
    打开文件数量的差异 由于pipe是半双工的传统IPC实现方式，所有两个线程通信需要两个pipe文件描述符，而用eventfd只需要打开一个文件描述符。 总所周知，文件描述符是系统中非常宝贵的资源，linux的默认值只有1024个而已，pipe只能在两个进程/线程间使用，面向连接，使用之前就需要创建好两个pipe,而eventfd是广播式的通知，可以多对多。
    内存使用的差别 eventfd是一个计数器，内核维护的成本非常低，大概是自旋锁+唤醒队列的大小，8个字节的传输成本也微乎其微，而pipe完全不同，一来一回数据在用户空间和内核空间有多达4次的复制，而且最糟糕的是，内核要为每个pipe分配最少4k的虚拟内存页，哪怕传送的数据长度为0.
    与epoll完美结合 eventfd设计之初就与epoll完美结合，支持非阻塞读取等，就是为epoll而生的,而pipe是Unix时代就有了，那时候不仅没有epoll,连linux还没诞生。
    当pipe只用来发送通知，放弃它，放心的使用eventfd。
    eventfd配合epoll才是它存在的原


IdleHandler
是MessageQueue的静态内部类
https://zhuanlan.zhihu.com/p/345819916
https://juejin.cn/post/6844904068129751047
执行的时机：messageQueue取消息的时候(next()函数), 没有msg或者取出的msg的执行时间大于当前时间的时候，就会遍历执行所有的idleHandler，执行完后返回next()中的for(;;)循环继续取消息
代码:
Message next() {
    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        ...
        nativePollOnce(ptr, nextPollTimeoutMillis);
        ...
        synchronized (this) {
            // 取消息
            ...

            // 没有msg或者取出的msg的执行时间大于当前时间的时候
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // No idle handlers to run.  Loop and wait some more.
                // 消息也不执行、idleHandler也没有的时候，内存记录下mBlocked = true 然后 返回循环
                mBlocked = true;
                continue;
            }

            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }
         for (int i = 0; i < pendingIdleHandlerCount; i++) {
            keep = idler.queueIdle();
            // 是否循环利用
            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }   
         // Reset the idle handler count to 0 so we do not run them again.
        pendingIdleHandlerCount = 0;

        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}
执行总结：
1. 在 MessageQueue 里 next 方法的 for 死循环内，获取 mIdleHandlers 的数量 pendingIdleHandlerCount；
2. 通过 mMessages == null || now < mMessages.when 判断当前消息队列为空或者目前没有需要执行的消息时，给 pendingIdleHandlerCount 赋值；
3. 当数量大于 0，遍历取出数组内的 IdleHandler，执行 queueIdle() ；
4. 返回值为 false 时，主动移除监听 mIdleHandlers.remove(idler) ；

使用场景：耗时短、不紧急的事情处理
1. 如果启动的 Activity、Fragment、Dialog 内含有大量数据和视图的加载，导致首次打开时动画切换卡顿或者一瞬间白屏，可将部分加载逻辑放到 queueIdle() 内处理。例如引导图的加载和弹窗提示等；
2. 系统源码中 ActivityThread 的 GcIdler，在某些场景等待消息队列暂时空闲时会尝试执行 GC 操作；
3. 系统源码中  ActivityThread 的 Idler，在 handleResumeActivity() 方法内会注册 Idler()，等待 handleResumeActivity 后视图绘制完成，消息队列暂时空闲时再调用 AMS 的 activityIdle 方法，检查页面的生命周期状态，触发 activity 的 stop 生命周期等。这也是为什么我们 BActivity 跳转 CActivity 时，BActivity 生命周期的 onStop() 会在 CActivity 的 onResume() 后。
4. 一些第三方框架 Glide 和 LeakCanary 等也使用到 IdleHandler，感兴趣的朋友可以看看源码



HandlerThread 
    HandlerThread extends Thread
// ***************************************** 
handlerThread怎么解决多线程问题的? 
case：主线程new了一个HandlerThread，然后调用start，然后紧接着new Hander(handlerThread.getLooper())，这里由于cpu时间片的原因，handlerThread的looper对象可能还没执行
ps：
public void run() {
    ...
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    ...
    Looper.loop();
    ...
}

/**
* This method returns the Looper associated with this thread. If this thread not been started
* or for any reason isAlive() returns false, this method will return null. If this thread
* has been started, this method will block until the looper has been initialized.  
* @return The looper.
*/
public Looper getLooper() {
    // If this thread not been started or for any reason isAlive() returns false，
    // case：如果handleThread没调用start，其他thread调用handleThread.getLooper()会返回null，不会堵塞
     if (!isAlive()) {
        return null;
    }
    
    // If the thread has been started, wait until the looper has been created.
    synchronized (this) {
        // 这里为什么用while而不是if呢？不确定其他上相同锁的对象地方有没有执行notifyAll(比如继承了handlerTahread去做些锁操作等)
        while (isAlive() && mLooper == null) {
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
    }
    return mLooper;
}
// ***************************************** 



okhttp：https://juejin.cn/post/6844904087788453896




android Application对象和多进程关系
同一个项目的FeelingApplication，AActivity、TestActivty、TestService，
在AndroidManifest.xml中配置如下
    其中TestActivity配置了android:process="com.Beggar.TestActivity"
    其中TestService配置了android:process="com.Beggar.TestService"
在代码里面：
    AActivity中启动TestActivity
    TestActivity中有一个static String flag = "1", 在onCreate中设置flag为2，然后启动TestService
日志如下：发现service的application是全新的对象，新进程对静态数据都是各自的数据互不影响
    I/FeelingApplication: attachBaseContext: com.li.feeling.init.application.FeelingApplication@c7c3095
    I/FeelingApplication: onCreate: com.li.feeling.init.application.FeelingApplication@c7c3095
    I/FeelingApplication: attachBaseContext: com.li.feeling.init.application.FeelingApplication@c7c3095
    I/FeelingApplication: onCreate: com.li.feeling.init.application.FeelingApplication@c7c3095
    I/FeelingApplication: TestActivity flag is: 2
    I/FeelingApplication: attachBaseContext: com.li.feeling.init.application.FeelingApplication@bbc984c
    I/FeelingApplication: onCreate: com.li.feeling.init.application.FeelingApplication@bbc984c
    I/FeelingApplication: TestService flag is: 1
多进程需要注意：由于Application的相关方法会被执行多次，因此一些功能的初始化方法要确保只调用一次。



ContentProvider
简单介绍的文章：https://blog.csdn.net/carson_ho/article/details/76101093
    定义：内容提供者，是android四大组建之一
    作用：进程间进行数据交互 & 共享，即夸进程通信
    原理：Binder
    优点：安全；访问简单、高效；统一了数据访问方式
ContentProvider 的常规用法是提供内容服务，而另一个特殊的用法是提供无侵入的初始化机制

ContentProvider.onCreate调用时机介于Application的attachBaseContext和onCreate之间
Application.attachBaseContext() --> installContentProviders(调用所有provider的onCreat方法) --> Application.onCreat() --> MainActivity


ANR
不同组件(Service、BroadCast、Provider、Input(5秒)等)的超时阀值各有不同(同一组建在前后台的都不一样)
case：App进程StartService方法，向SystemServer进程请求创建Service，此时SystemServer进程的Binder对象向ActivityManager埋下定时炸弹，然后该binder请求scheduleCreateService
    去创建Service进程，当service创建完毕的时候，请求ActivityManager拆除定时炸弹


lifeCycle
1. lifeCycleRegistry添加观察者(LifeCycleObserver)的时候，会先同步下观察者的状态
2. LifeCycleObserver一般有两种，注解事件方法和FullLifecycleObserver，注解是运行时，添加observer的时候会动态解析注解，方法和事件注解一一对应Map<MethodReference, Lifecycle.Event>



android数据恢复：https://juejin.cn/post/6844904079265644551

在 Android 系统中，需要数据恢复有如下两种场景：
场景1：资源相关的配置发生改变导致 Activity 被杀死并重新创建。(不考虑在清单文件中配置 android:configChanges 的特殊情况)
场景2：资源内存不足导致低优先级的 Activity 被杀死。
解决方案：
1. 使用 onSaveInstanceState 与 onRestoreInstanceState
2. 使用 Fragment 的 setRetainInstance
3. 使用 onRetainNonConfigurationInstance 与 getLastNonConfigurationInstance (ViewModel的恢复就是采用这种方式的)

onSaveInstanceState不是每次都会调用(如用户手动按下返回键来关闭activity)，调用时机：
如果被调用，对于以 Build.VERSION_CODES.P 开头的平台为目标的应用程序，此方法将在 onStop 之后发生。对于针对早期平台版本的应用程序，此方法将在 onStop 之前发生，并且无法保证它是在 onPause 之前还是之后发生。
在onCreate和onRestoreInstanceState中都可以拿到onSaveInstanceState不存储的bundle(函数参数的形式)
onRestoreInstanceState调用时机：
这个方法在 onStart 和 onPostCreate 之间被调用。只有在重新创建活动时才会调用此方法；如果出于任何其他原因调用 onStart，则不会调用该方法。


ViewModel的好文章：https://juejin.cn/post/6844904079265644551


RecyclerView
局部刷新：https://juejin.cn/post/6844903817130016782
四级缓存：https://www.jianshu.com/p/3e9aa4bdaefd
Scrap(屏幕内的，可以拿来直接使用，如局部刷新时)：Recycler.mAttachedScrap
Cache()：Recycler.mCachedViews
ViewCacheExtension
RecycledViewPool

缓存使用
onTouchEvent方()中move事件-->scrollByInternal()-->scrollStep()-->
layoutManager中的：  LayoutManager.scrollVerticallyBy() ｜ scrollHorizontallyBy()-->scrollBy()-->fill()-->layoutChunk()-->
LayoutState中的：    layoutState.next(recycler)-->
Recycler中的：       recycler.getViewForPosition(mCurrentPosition)-->tryGetViewHolderForPositionByDeadline(position)-->先查mChangedScrap中-->开始走4级缓存
注意从pool中取到ViewHolde后，会调用vh.resetInternal()去清空数据
如果缓存中拿不到，就走create了


Fragment懒加载(主要分析ViewPager场景下)，很不错的文章：https://juejin.cn/post/6844904050698223624

ViewPager2和viewPager
1. ViewPager2是RecyclerView那一套
2. ViewPager2默认是懒加载的，ViewPager默认是预加载的(左右各1个)

ViewPager
为什么ViewPager设置宽度、高度无效？
在OnMeasure()中，先立刻执行了setMeasuredDimension(getDefaultSize(0, widthMeasureSpec),getDefaultSize(0, heightMeasureSpec)); 并没有测量完所有的孩子后根据孩子的大小设置自己的。
可见，ViewPager的宽高是由它的容器决定的。
官方注释这么做的原因：我们依靠容器来指定视图的布局大小。我们无法真正知道它是什么，因为我们将添加和删除不同的任意视图并且不希望布局在这种情况下发生变化。

默认缓存数量为左右个一个: DEFAULT_OFFSCREEN_PAGES = 1
void setOffscreenPageLimit(int limit) {
    if (limit < DEFAULT_OFFSCREEN_PAGES) {
            limit = DEFAULT_OFFSCREEN_PAGES;
        }
        if (limit != mOffscreenPageLimit) {
            mOffscreenPageLimit = limit;
            populate();
        }
}

ArrayList<ItemInfo> mItems; // 缓存的页面
populate(int newCurrentItem)函数， 还有个void populate() { populate(mCurItem); }函数
    1. 如果currentItem为空(比如第一次显示的时候)，那么调用IteamInfo addNewItem(int position, int index)函数，内部是mAdapter.instantiateItem(this, position)
    2. 紧接着判断如果currentItem不为空，然后起两个循环，分别检验curItem的左右两侧的，
        (1)如果要销毁的，那么执行mItems.remove(itemIndex) 和 mAdapter.destroyItem()
        (2)如果要缓存，那么调用IteamInfo addNewItem(int position, int index)函数，实例化出来item，函数内部会调用mAdapter.instantiateItem 和 mItems.add(xx)

PagerAdapter mAdapter; PagerAdapter的两个直接子类FragmentStatePagerAdapter和FragmentPagerAdapter
FragmentPagerAdapter中destroyItem()方法会执行FragmentTransaction.detach(fragment)方法，仅销毁视图，并不会销毁fragment实例
FragmentStatePagerAdapter中destroyItem()方法会执行FragmentTransaction.remove(fragment)方法，会彻底将fragment从当前Activity的FragmentManager中移除，
    state表明销毁时，会将其onSaveInstanceState(Bundle outState)中的bundle信息保存下来，当用户切换回来，可以通过该bundle恢复生成新的fragment，也就是说，你可以在onSaveInstanceState(Bundle outState)方法中保存一些数据，在onCreate中进行恢复创建。
        在destroyItem的时候会保存state(关联position)，instantiateItem方法中会从拿出保存的state去创建fragment

adapter原理：
准备适配             void startUpdate(ViewGroup container)           如ViewPager在populate方法开始的时候调用
创建item            Object instantiateItem(ViewGroup container, int position)            如ViewPager在populate中
                                                     FragmentPagerAdapter实现           实现为先通过mFragmentManager.findFragmentByTag(name)找，name是postion加一些字符拼接的，如果能拿到那么fragmentTraction.attach
                                                                                        没有的话通过getItem(position)拿，然后fragmentTraction.add
                                                     FragmentStatePagerAdapter实现     
                                                                                        先通过 mFragments.get(position)拿，有的话直接return
                                                                                        没有的话通过getItem(position)拿，然后mFragments.set(position, fragment)，然后fragmentTraction.add。         
销毁item            void destroyItem(ViewGroup container, int position, Object object)   如ViewPager在populate中，
                                                      FragmentPagerAdapter的实现        fragmentTraction.detach，
                                                      FragmentStatePagerAdapter的实现   fragmentTraction.remove，ps：mFragments.set(position, null);
设置当前的item       void setPrimaryItem(ViewGroup container, int position, Object object) viewPager在得到当前item且创建销毁并缓存好item后，调用
                                                                                    会让旧的currentFragment.setUserUserVisibleHint(flase)
                                                                                    让要设置的fragment.setUserUserVisibleHint(true) 和 fragment.setUserUserVisibleHint(flase) 
完成适配             void finishUpdate(ViewGroup container)      如viewPager在populate方法快结束的时候调用，FragmentPagerAdapter的实现为fragmentTraction.commitNowAllowingStateLoss



WindowManager

Window, WindowManager、WindowManagerGlobal、WindowManagerService之间的关系：https://juejin.cn/post/6844903893634138120

Window分类：
1. ApplicationWindow：
    Activity的Window类型是TYPE_APPLICATION，在WMS有一个唯一的AppWindowToken与之对应。
    Dialog的Window类型也是TYPE_APPLICATION(Dialog不是子窗口)，Dialog和其所在的Activity共用一个AppWindowToken
2. SubWindow 
    子窗口，不能独立存在，必须依附其他窗口(共用一个token)。
    PopupWindow的Window类型是TYPE_APPLICATION_PANEL，TYPE_APPLICATION_PANEL是子窗口的一种,PopupWindow和其所在的Activity共用一个AppWindowToken
3. SystemWindow 
    toast、输入法窗口、系统音量条窗口、系统错误窗口都属于系统窗口
    Toast的Window类型是TYPE_TOAST，TYPE_TOAST是系统窗口的一种，Toast的token是由NotificationManagerService创建的，每个Toast对应一个自己的token，Toast的token在WMS对应的是WindowToken。

window类型的type值：值越大，展示的越靠前(屏幕最上面)，比如来了个电话弹窗会盖在最上面
    ApplicationWindow：FIRST_APPLICATION_WINDOW = 1  LAST_APPLICATION_WINDOW = 99
    SubWindow：FIRST_SUB_WINDOW = 1000  LAST_SUB_WINDOW = 1999;
    SystemWindow：FIRST_SYSTEM_WINDOW = 2000  LAST_SYSTEM_WINDOW = 2999

Activity/Dialog/PopupWindow/Toast与WindowState:
    Activity/Dialog/PopupWindow/Toast在WMS都有对应的WindowState，
    只是Activity/Dialog/PopupWindow的WindowState属于同一个AppWindowToken，也就是Activity的token,
    而Toast的WindowState属于自己独有的WindowToken。

Window的flag：一些属性，比如脸靠近屏幕时不能触摸事件等flag

// 继承自ViewManager，ViewGroup也实现了ViewManager
public interface WindowManager extends ViewManager {

}
//  实现类是WindowManagerImpl
public final class WindowManagerImpl implements WindowManager {
}

ViewRootImpl
public final class ViewRootImpl implements ViewParent,View.AttachInfo.Callbacks, ThreadedRenderer.DrawCallbacks { }
1. View树的树根，并管理view树
2. 触发view的测量、布局和绘制
3. 输入响应的中转站：
    ViewRootImpl --> DecorView --> Activity --> PhoneWindow --> DecorView --> ViewGroup --> View
    为什么DecorView不直接把事件分发给子view呢？因为窗口要有一些处理，比如打电话脸部靠近屏幕的时候不会响应点击；在比如dialog点击外部区域收起弹窗
4. 负责与wms进行进程间通信： (ViewRootImpl-->IWindowSession)本地进程  --------Binder通信--------  SystemService进程(WMS的Session)


window创建和显示的过程
大流程简单总结：
1. ActivityThread.performLaunchActivity() --> Activity.attach() 
    Activity#attach() 方法内phontWindow被创建，并同时创建WindowManagerImpl负责维护phoneWindow的内容
2. ActivityThread.mInstrumentation.callActivityOnCreate() --> Activity.setContentView()
    Activity#setContentView() 方法内创建DecorView作为phoneWindow的内容(phoneWindow.installDecor函数里面会decorView.setWindow(this))
3. ActivityThread.handleResumeActivity() --> 
                                      -->  WindowManagerImpl.addView()
                                      -->  WindowManagerGlobal.addView()
                                      -->  ViewRootImpl.addToDisplay()
                                      -->  Session.addWindow()
                                      -->  WMS
    WindowManagerImpl决定管理DecorView，并创建ViewRootImpl实例，将ViewRootImpl与View树关联，这样ViewRootImpl就可以指挥view树的具体工作。

Window更新的过程
windowManagerImpl.updateViewLayout(view, params) 
    -> WindowManagerGlobal.updateViewLayout(view, params) 
        --> view.setLayoutParams(wparams)
        --> viewRootImpl.setLayoutParams(wparams, false) 
            --> viewRootImpl.scheduleTraversals()


window创建和显示的过程 -- 代码细节总结：
1. create
ActivitThread: performLaunchActivity()
                    --> ContextImpl appContext 创建 
                    --> activity = mInstrumentation.newActivity() 
                    --> appContext.setOuterContext(activity);
                    --> activity.attach(appContext, ..., activityClientRecord.token, ..., r.parent, ...)
 Activity:              --> mWindow = new PhoneWindow(this, window, activityConfigCallback)
                        --> mToken = token // 这个token是activity被创建的时候，在ams侧生成的，然后传递到wms侧以及app端，参考 https://www.jianshu.com/p/1422c02da9da
                        --> mWindow.setWindowManager((WindowManager)context.getSystemService(Context.WINDOW_SERVICE), mToken, ...) // activity的context重写了getSystemService(Context.WINDOW_SERVICE)
 Window:                    --> mAppToken = appToken
                            --> mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
                                --> return new WindowManagerImpl(mContext, parentWindow);
                        --> mWindow.setContainer(mParent.getWindow());
                        --> mWindowManager = mWindow.getWindowManager();
                    --> mInstrumentation.callActivityOnCreate

2. resume
ActivitThread:  handleResumeActivity()
                     --> performResumeActivity()
                        --> activity.performResume
Activity:                --> mInstrumentation.callActivityOnResume
                            --> dispatchActivityPostResumed(); 这里面就会分发activity的resume事件
                                --> ((Application.ActivityLifecycleCallbacks) callbacks[i]).onActivityPostResumed(this)
                                --> getApplication().dispatchActivityPostResumed(this);
                     --> r.window = r.activity.getWindow();
                         View decor = r.window.getDecorView();
                         decor.setVisibility(View.INVISIBLE);
                         ViewManager wm = a.getWindowManager(); // 这里的返回的实例是WindowManagerImpl
                         WindowManager.LayoutParams l = r.window.getAttributes(); // 如果不设置的话，LayoutParams的默认宽高都是match_parent
                         wm.addView(decor, l);
WindowMangerImpl：          --> WindowManagerGlobal.getInstance().addView(view, params,mContext.getDisplayNoVerify(), mParentWindow,mContext.getUserId()) // 注意这里的parentWindow, activity的话传入的是自己的phoneWindow
WindowManagerGlobal:           --> 如果parentWindow != null的话，parentWindow.adjustLayoutParamsForSubWindow(params)
Window：                            --> 在这里根据window的类型(subWindow、systemWindow、其他，注意type默认是TYPE_APPLICATION)做一些处理
                                        1. 如果是subWindow, 那么wp.token = decor.getWindowToken()
                                        2. 如果是systemWindow，xxxxx
                                        3. else，wp.token = mContainer == null ? mAppToken : mContainer.mAppToken; // mAppToken是activity的mToken，mContainer见activity.Attach中mWindow.setContainer(mParent.getWindow());
                                    --> 其他的一些操作，比如wp.packageName = mContext.getPackageName();，
                                        在比如设置硬件加速wp.flags |= FLAG_HARDWARE_ACCELERATED;
                               --> 如果已经添加过这个view了(mViews)，那么抛异常："View " + view + " has already been added to the window manager."
                               --> ViewRootImpl root = new ViewRootImpl(view.getContext(), display);
                                    --> 构造函数中 mAttachInfo = new View.AttachInfo(mWindowSession, mWindow, ...)
                               --> view.setLayoutParams(wparams); // 给根view设置LayoutParams
                               -->  mViews.add(view); mRoots.add(root); mParams.add(wparams); // 可见WindowManagerGlobal是管理一个进程的所有的view
                               --> root.setView(view, wparams, panelParentView, userId); // 到这里view的测量渲染等全部交给ViewRootImpl去处理了
ViewRootImpl:                    void setView(View view, WindowManager.LayoutParams attrs, View panelParentView, int userId) 
                                    --> requestLayout() // 这是第一次view触发绘制的地方
                                        --> checkThread(); // 检查线程，也就是子线程无法更新ui的原因。
                                        --> mLayoutRequested = true; // 在measureHierarchy()后，设置为false。然后开始所谓的3大流程
                                        --> scheduleTraversals(); // invalidate中先mDirty.set(0, 0, mWidth, mHeight);然后调用scheduleTraversals()，注意这里没有给mLayoutRequested设置为true
                                            点比较多，单独分析
                                    --> InputChannel inputChannel = new InputChannel();  // 监听事件
                                        mWindowSession.addToDisplayAsUser(mWindow, ..., inputChannel, ...)  // 把窗口添加到wms上
                                        mInputEventReceiver = new WindowInputEventReceiver(inputChannel, Looper.myLooper()); WindowInputEventReceiver是ViewRootImpl的内部类
                                    --> view.assignParent(this); // 设置父亲为VRI: view.mParent = parent（类型是ViewParent，如ViewGroup实现了ViewParent）


ViewRootImpl.scheduleTraversals()： https://cloud.tencent.com/developer/article/1582588
    --> mTraversalScheduled = true;
    --> mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier(); // 添加同步屏障
    --> mChoreographer.postCallback(Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null); // 编舞者
        --> Choreographer#scheduleFrameLocked(now);
            --> scheduleVsyncLocked() // 请求Vsync垂直同步信号
                --> mDisplayEventReceiver.scheduleVsync(); --> nativeScheduleVsync(mReceiverPtr);
        --> messageQueue.next() --> messageQueue.nativePollOnce()
            --> native调用mDisplayEventReceiver.dispatchVsync()  // 被到来的Vsync唤醒 
                --> onVsync(long timestampNanos, long physicalDisplayId, int frame) // 执行vsync回调
                    --> 向messageQueue插入异步消息，消息的执行体是Choreographer.doFrame(mTimestampNanos, mFrame);
                        --> 内部如果丢帧会报经典的日志："Skipped " + skippedFrames + " frames!  "+"The application may be doing too much work on its main thread."
                            doFrame() {
                                ......
                                // 当前时间
                                startNanos = System.nanoTime();
                                // 经过了多长时间
                                final long jitterNanos = startNanos - frameTimeNanos;
                                // 超过了间隔(比如16ms), 从申请vsync到这里执行应该在mFrameIntervalNanos间隔内
                                if (jitterNanos >= mFrameIntervalNanos) {
                                    // 计算丢了多少帧
                                    final long skippedFrames = jitterNanos / mFrameIntervalNanos;
                                    log("Skipped xxxx")
                                }
                                ......
                            }
                    --> messageQueue取出该消息后，执行message，即doFrame，内部执行ViewRootImpl向mChoreographer添加的mTraversalRunnable，该runnable内执行ViewRootImpl#doTraversal
                     doTraversal函数中：
                        --> mTraversalScheduled = false; mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier); // 移除同步屏障
                        --> performTraversals();
                            --> measureHierarchy() // 预测量，内部最多执行3次performMeasure()
                            --> performMeasure() // measureHierarchy()后，performMeasure()还最多会执行两次
                                --> measure()
                            --> performLayout()
                                --> layout()
                            --> performDraw()
                                --> draw()




view measure
1. MeasureSpec.UNSPECIFIED用在啥场景：
    MeasureSpec.UNSPECIFIED 表示父 View 对子 View 的大小没有任何限制，子 View 可以任意取大小。在这种情况下，子 View 的宽度和高度可以是任意值（可以是 0，也可以是非常大或非常小），开发者可以根据需要自由地计算自己的大小。
    MeasureSpec.UNSPECIFIED 主要用于一些特殊的布局，通常是自定义 View 或一些高度自适应的控件中。比如，当我们需要实现一个可以无限滚动的视图，我们就需要在onMeasure()方法中根据MeasureSpec.UNSPECIFIED测量子 View 的大小，然后根据子 View 的大小和数量计算出整个布局的大小。
    另外，对于一些不会限制子 View 大小的布局，比如 FrameLayout，通常也会将 MeasureSpec 的模式设置为 UNSPECIFIED。这样一来，子 View 可以占据 FrameLayout 的全部空间，从而避免了布局出现“空洞”的情况。
2. 为什么父亲向孩子传递的measureSpec要在父View中结合子View一起计算在传递给子View呢，而不是直接传递父View的measureSpec，然后子View自己根据自己的LayoutParams计算出自己的measureSpec呢
    1. 因为父亲的限制是多种多样的，父view有自己的限制规则，这个规则没办法传递给子View的。
    2. 两个ViewGroup的限制规则不一样，然后A这个子view要添加进这两个viewGroup，子view的measureSpec怎么计算呢？因此只能父亲计算的
    3. 难道每新定义一个子view的时候，这个计算measureSpec的都要在写一遍吗？


view layout
一些位置相关的函数
    getLeft()：左边距离父亲左侧的距离
    getRight(): 右边距离父亲左侧的距离
    getTop(): 上边距离父亲上侧的距离
    getBottom(): 下边距离父亲上侧的距离
    getX() getY() 以父亲左上角为原点
    getRawX() getRawY() 以屏幕原点(左上角)的坐标

getMeasureWidth() : 在measure()过程结束后就能获取，通过setMeasuredDimension()方法来设置
getWidth(): 在Layout()过程结束后才能获取，通过视图右边的坐标-左边的坐标得到：mRight-mLeft


requestLayout() 和 invalidate区别以及原因：

ViewGroup为什么默认不会执行onDraw()? 
结论：
1. decorView的onDraw会执行
2. ViewGroup的WILL_NOT_DRAW标志位默认就是true的，
     private void initViewGroup() {
        .......
        if (!isShowingLayoutBounds()) {
            setFlags(WILL_NOT_DRAW, DRAW_MASK);
        }
        .........
    }
    当WILL_NOT_DRAW标志时，且没有设置背景等，那么会设置PFLAG_SKIP_DRAW标记位，就不会走draw(canvas)，然后就不会走onDraw()
    void setFlags(int flags, int mask) {
        ........
        if ((changed & DRAW_MASK) != 0) {
            if ((mViewFlags & WILL_NOT_DRAW) != 0) {
                if (mBackground != null
                        || mDefaultFocusHighlight != null
                        || (mForegroundInfo != null && mForegroundInfo.mDrawable != null)) {
                    mPrivateFlags &= ~PFLAG_SKIP_DRAW;
                } else {
                    mPrivateFlags |= PFLAG_SKIP_DRAW;
                }
            } else {
                mPrivateFlags &= ~PFLAG_SKIP_DRAW;
            }
            requestLayout();
            invalidate(true);
        }
        .........
    }
3. 如果设置了BackgroundDrawable、Foreground等，那么会设置~PFLAG_SKIP_DRAW, 就会走draw(canvas),然后就会走onDraw()

因此，想让viewGroup走绘制draw的话，要调用View.setWillNotDraw(false);
public void setWillNotDraw(boolean willNotDraw) {
        setFlags(willNotDraw ? WILL_NOT_DRAW : 0, DRAW_MASK);
}

View.draw(canvas) (decorView)
    --> drawBackground(canvas);
    --> onDraw(canvas)
    下面的流程是个递归操作
    --> dispatchDraw(canvas); // draw the children (实现是ViewGroup.dispatchDraw())
            --> drawChild(canvas, child, drawingTime) // 函数的实现是child.draw(canvas, this, drawingTime);
                    --> View.draw(Canvas canvas, ViewGroup parent,  long drawingTime) // 注意调用的3参的draw
                            --> renderNode = updateDisplayListIfDirty();
                                    // ~PFLAG_SKIP_DRAW这个标记位什么时候被设置呢？
                                    // 比如view.setBackgroundDrawable()、setForeground()等
                                    // 另外，WILL_NOT_DRAW标志时，且没有设置背景等，那么会设置PFLAG_SKIP_DRAW标记位
                                    --> 如果设置了PFLAG_SKIP_DRAW这个标记位的话，那么执行 dispatchDraw(canvas); // draw the children  (实现是ViewGroup.dispatchDraw())
                                    --> 否则执行 draw(canvas);


SurfaceFlinger
SF是整个Android系统渲染的核心进程，所有应用的渲染逻辑最终都会来到SF中进行处理，最终会把处理后的图像数据交给CPU或者GPU处理。
SF是生产者消费者思想，生产者生产图元添加到SF的队列中，SF消费队列的图元数据(绘制对应的Layer)
View、Canvas与Surface的关系：https://www.jianshu.com/p/6d99948d2a0a
DecorView --- Surface(canvas = surface.lockCanvas(Rect dirty))，存储图像数据 --- Layer图层     // 3者一一对应
......
DecorView --- Surface，存储图像数据 --- Layer图层
SurfaceFlinger把多个图层合成，然后让cpu或者gpu处理

好文章：https://www.jianshu.com/p/3c61375cc15b
每一个应用程序的图层在SurfaceFlinger里称为一个Layer， 而每个Layer都拥有一个独立的BufferQueue, 每个BufferQueue都有多个Buffer,Android 系统上目前支持每个Layer最多64个buffer, 这个最大值被定义在frameworks/native/gui/BufferQueueDefs.h， 每个buffer用一个结构体BufferSlot来代表。
<img width="813" alt="image" src="https://user-images.githubusercontent.com/49143666/231415156-2d4a87fc-f5d1-4a23-95b8-b1e749d68b1b.png">

    
<img width="964" alt="image" src="https://user-images.githubusercontent.com/49143666/231407310-43f56f2e-cde0-4e90-8714-2b977639314b.png">    
双缓冲区 + 同步信号 
如果没有同步信号的话：本质就是两个缓冲区操作速率不一样
    1. 系统帧率小于屏幕刷新率：长时间都是同一帧，感觉卡顿
    2. 系统帧率大于屏幕刷新率：屏幕撕裂，甚至跳帧
    方案：让屏幕控制前后缓冲区的切换，让系统帧率配合屏幕刷新率的节奏
<img width="625" alt="image" src="https://user-images.githubusercontent.com/49143666/231373888-a3d8c49b-fa24-4319-a0c3-25d95642b8a1.png">
<img width="530" alt="image" src="https://user-images.githubusercontent.com/49143666/231373952-fd371dbf-f3c6-40ca-a109-93b7631bb0ee.png">
<img width="855" alt="image" src="https://user-images.githubusercontent.com/49143666/231375187-3b273f61-0c71-4456-84ad-03f315b5e935.png">
图中CPU比如生产位图（BitMap），然后GPU负责颜色转换(如#FFF999D转为LED的硬件色)、珊格化(如位图大小和要展示(屏幕)的大小不一样，要缩放，gpu就计算放大缩小)

双缓冲的问题：CPU、GPU空等(图中的空白部分都在等待)
<img width="973" alt="image" src="https://user-images.githubusercontent.com/49143666/231382329-24efa3ea-d26a-4a08-89d9-8ae292671685.png">

三缓冲：
<img width="896" alt="image" src="https://user-images.githubusercontent.com/49143666/231383626-8671ed39-37a5-412c-aa57-fb52e064a105.png">
每个window都有3个缓冲：查看命令 adb shell dumpsys SurfaceFlinger
<img width="1715" alt="image" src="https://user-images.githubusercontent.com/49143666/231386229-f0489ec1-f778-4c34-9e65-58472e40a871.png">
<img width="1532" alt="image" src="https://user-images.githubusercontent.com/49143666/231386453-3b17b819-9ace-45ab-a6d5-fb56a226889d.png">


App如何与SurfaceFlinger通信？
<img width="1288" alt="image" src="https://user-images.githubusercontent.com/49143666/231704620-655d804c-53c0-403a-9590-048a4753882d.png">

View的绘制如何把数据传递给SurfaceFlinger？
A：先surface.lockCanvas()申请buffer，然后绘制，绘制完了调用surface.unlockCanvasAndPost()。这里surface是有surfafceFlinger的bufferQueue的生产者的本地代理

 
    
事件分发：
InputManagerService: https://www.jianshu.com/p/f05d6b05ba17

三块代码：
----------------------------------------------
第一块代码：是否拦截子view，intercepted为true拦截，为false不拦截
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
        || mFirstTouchTarget != null) {

} else {

}

第二块代码：遍历子view，询问子view是否要处理事件
if (!canceled && !intercepted) {

}

第三块代码
1. 如果子view都不处理，询问自己是否要处理事件
2. 子view处理 ---
----------------------------------------------

处理的事件： Action_down、Action_move、.......Action_move、Action_up
down事件谁处理的，move事件也是谁处理

左右滑动的ViewPager中嵌套上下滑动的ListView
1. ViewPager.onInterceptTouchEvent 为 true   --》 上下滑动不可以，左右可以
2. ViewPager.onInterceptTouchEvent 为 false  --》 上下滑动可以，左右不可以
3. ViewPager.onInterceptTouchEvent 为 false，
    ListView重写dispatchTouchEvent返回false   -->  上下滑动不可以，左右可以

第2点：ViewPager.onInterceptTouchEvent 为 false  --》 上下滑动可以，左右不可以
答：Action_Down -- 询问子view是否处理事件，只在Down的时候处理 -- target.child = ListView
   Action_Move -- target.child - ListView
   dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign) -- 孩子(ListView)处理事件

第3点：ViewPager.onInterceptTouchEvent 为 false，
    ListView重写dispatchTouchEvent返回false   -->  上下滑动不可以，左右可以
答：down事件，询问listView处理这个事件吗？
  ListView重写dispatchTouchEvent返回false，导致if进不去：dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)
  viewPager的所有孩子都不处理事件，所以自己处理事件，左右可以滑动：进入if
    if (mFirstTouchTarget == null) {
        handled = dispatchTransformedTouchEvent(ev, canceled, null,TouchTarget.ALL_POINTER_IDS);
    }

处理事件冲突：
1. 内部拦截法(由子view根据条件来让事件由谁处理) -- 一定想办法让子view拿到事件
2. 外部拦截法(由父view。。。。)






View
自定义view的3个构造方法，xml文件被LayoutInflater解析的时候，如果那几个factory没有创建出view的话，最终会反射调用两参的构造方法，且构造器会缓存到layoutInfater中:HashMap<viewName, constructor>

ViewStub: 继承自View
通过setVisibility(VIsible) 或者 inflate()方法可以展示子view
原理：
1. 在构造方法里面setVisibility(GONE);
2. 重写了onMeasure方法，方法的实现是setMeasuredDimension(0, 0);
3. setVisibility()重写了，不会直接设置viewStub本身，而是设置要替换的view的显隐性，如果view还没infalte，那么inflate出来
    (1) viewStub里面有个field:WeakReference<View> mInflatedViewRef, 弱引用inflate出的view
4. inflate()方法，
    (1) 先拿到viewStub的parent，
    (2) 然后把要替换的布局infalte出来, 如果ViewStub设置了id的话，那么把替换的view的id设置为viewStub的id (注意如果子布局设置了id那么会找不到这个id)
    (3) viewStub的parent删除viewstub这个子view，然后把替换的view添加进去。


include标签：直接在LayoutInflater.parseInclude函数添加替换的layout的
1. 不能作为根节点
2. <Include> 标签中设置的宽高等信息，会变成LayoutParams对象设置给inflate出来的view
3. 如果Include设置了id的话，那么把替换的view的id设置为Include的id
4. 要替换的layout获取AttributeSet childAttrs，然后递归去inflateChildren
5. parent去addView(inflate出来的view)



Surface：
https://blog.csdn.net/m0_37673128/article/details/113265492
https://kstack.corp.kuaishou.com/article/654


android资源加载：
Activity的Resources： getResource().getDrawable(drawableId)；
    ActivityThread: performLaunchActivity() --> ContextImpl appContext = createBaseContextForActivity(ActivityClientRecord r) 注意这时activity还没被new出来
            --> ContextImpl appContext = ContextImpl.createActivityContext(ActivityThread mainThread, LoadedApk packageInfo, ActivityInfo activityInfo, IBinder activityToken, .....)
    ContextImpl:--> ContextImpl context = new ContextImpl(....) --> context.setResources(resourcesManager.createBaseTokenResources(...)
    ResourcesManager: -->createResourcesForActivity() --> ResourcesImpl resourcesImpl = findOrCreateResourcesImplForKeyLocked(...)
                        -->ResourcesImpl createResourcesImpl(...)-->AssetManager assets = createAssetManager(...) --> apkAssets = ApkAssets.loadFromPath(key.path...) 这个path就是res目录，AssetManager就是通过ApkAssets构建的
                        --> ResourcesImpl impl = new ResourcesImpl(assets, ...)

Dialog的Resources：没看代码，但是我们知道Dialog构造的时候需要传递Activity的Context

app换肤的思路：参考的这个库的https://github.com/ximsfei/Android-skin-support
1. 知道xml的view如何解析的
2. 如何拦截系统的创建流程？LayoutInflater.setFactory2可以拦截--aop的思路去实现(监听activity的create)
3. 拦截怎么做？重写系统的创建过程代码(copy源码)
4. 收集view以及属性
5. 创建皮肤包 apkAssets
6. 如何使用？只用插件(apk)的res 
    (1) 系统的资源是如何加载的
    (2) 通过hook技术，单独创建一个AssetManager专门去加载皮肤包的资源
    (3) 通过反射addAssetPath方法放入皮肤包的路径 从而得到 加载皮肤包资源的AssetManager
    (4) 通过app的资源id --》找到app的资源的name和type --》找到皮肤包中对应资源的id


app页面置灰：
描述一个颜色的时候，会有如下几个数据：色相、饱和度、亮度/明度。
//  色相：色彩的基本属性，就是平常所说的颜色名称，如红色、黄色等。
//  饱和度：是指色彩的纯度，越高色彩越纯，低则逐渐变灰，取值0-100%。
//  亮度/明度：色彩的明亮程度，一般情况下颜色加白色亮就越来越高，加黑色则越来越暗，取值0-100%。

Paint()对象的饱和度设置为0即可变成灰色：
private val garyPaint: Paint by lazy {
    Paint().apply {
      val garyColorMatrix = ColorMatrix()
      // 设置饱和度为0
      garyColorMatrix.setSaturation(0f)
      colorFilter = ColorMatrixColorFilter(garyColorMatrix)
    }
}
fun garyActivity(activity: Activity) {
    garyView(activity.window.decorView)
}
fun garyView(view: View) {
    // LAYER_TYPE_HARDWARE和LAYER_TYPE_SOFTWARE都可以
    view.setLayerType(View.LAYER_TYPE_HARDWARE, garyPaint)
}


Toast:
toast必须在主线程吗？ https://juejin.cn/post/6844904103538065422
如果直接new一个线程，不loop的话，调用Toast.makeText()，该方法里面会取looper，拿不到的话会抛异常Can't toast on a thread that has not called Looper.prepare()
Toast展示内容，是inflate一个布局(主体就是TextView)，然后拿到INotificationManager服务，入队







Android startup.Initializer :



leakCanary的内存泄漏分析过程
1)注册监听Activity生命周期onDestroy事件
(2)在Activity onDestroy事件回调中创建KeyedWeakReference对象，并关联ReferenceQueue
(3)延时5秒检查目标对象是否回收
(4)未回收则开启服务，dump heap获取内存快照hprof文件
(5)解析hprof文件根据KeyedWeakReference类型过滤找到内存泄漏对象
(6)计算对象到GC roots的最短路径，并合并所有最短路径为一棵树
(7)输出分析结果，并根据分析结果展示到可视化页面


编码、字符集、讲一下 Unicode 和 UTF-8 的区别：https://zhuanlan.zhihu.com/p/51828216

protocol Buffer原理：https://juejin.cn/post/6844903997292150791
为什么Protocol Buffer更快，更小，这里再总结一下：
1. 序列化的时候，不序列化key的name，只序列化key的编号
2. 序列化的时候，没有赋值的key，不参与序列化，反序列化的时候直接使用默认值填充
3. 可变长度编码，减小字节占用
4. TLV编码，去除没有的符号，使数据更加紧凑





LeakCanary库的代码看下
livedata mvvm
jvm内存、垃圾回收
类加载
lru、lfu
view渲染
binder
动画
okhttp、fresco、retrofit、gson、rxjava、ARouter
热修复的几种方法
插件化的几种方法
viewFlipper、viewPager看一下


