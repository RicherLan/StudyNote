## 线程池
#### 线程池的状态，6种
RUNNING：运行状态，线程池创建好之后就会进入此状态，如果不手动调用关闭方法，那么线程池在整个程序运行期间都是此状态。
SHUTDOWN：关闭状态，不再接受新任务提交，但是会将已保存在任务队列中的任务处理完。
STOP：停止状态，不再接受新任务提交，并且会中断当前正在执行的任务、放弃任务队列中已有的任务。
TIDYING：整理状态，所有的任务都执行完毕后（也包括任务队列中的任务执行完），当前线程池中的活动线程数降为 0 时的状态。到此状态之后，会调用线程池的 terminated() 方法。
TERMINATED：销毁状态，当执行完线程池的 terminated() 方法之后就会变为此状
```java
public class ThreadPoolExecutor extends AbstractExecutorService {
     // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
}
```
* 线程池状态转移
线程池的状态转移有两条路径：
当调用 shutdown() 方法时，线程池的状态会从 RUNNING 到 SHUTDOWN，再到 TIDYING，最后到 TERMENATED 销毁状态。
当调用 shutdownNow() 方法时，线程池的状态会从 RUNNING 到 STOP，再到 TIDYING，最后到 TERMENATED 销毁状态。
![image](https://github.com/BeggarLan/StudyNote/assets/49143666/3293eb83-5d52-45ec-bb8e-b04135559f41)



## 线程、锁：

#### 线程状态
* java中线程的状态分为6种。
1. 初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。
2. 运行(RUNNABLE)：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
3. 阻塞(BLOCKED)：表示线程阻塞于锁。
4. 等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
5. 超时等待(TIMED_WAITING)：该状态不同于WAITING，它可以在指定的时间后自行返回。
6. 终止(TERMINATED)：表示该线程已经执行完毕。
![image](https://github.com/BeggarLan/StudyNote/assets/49143666/64808e68-a0cc-476a-b1c0-b26ed5419082)


#### suspend和stop弃用原因：
1.suspend()，在调用该方法暂停线程的时候，线程由running状态变成blocked，需要等待resume方法将其重新变成runnable。
而线程由running状态变成blocked时，只释放了CPU资源，没有释放锁资源，可能出现死锁。
比如：线程A拿着锁1被suspend了进入了blocked状态，等待线程B调用resume将线程A重新runnable。但是线程B一直在lock pool中等待锁1,线程B要拿到锁1才能running去执行resumeA。这就死锁了。

2.stop()，调用stop方法无论run()中的逻辑是否执行完，都会释放CPU资源，释放锁资源。这会导致线程不安全。
比如：线程A的逻辑是转账（获得锁，1号账户减少100元，2号账户增加100元，释放锁），那线程A刚执行到1号账户减少100元就被调用了stop方法，释放了锁资源，释放了CPU资源。1号账户平白无故少了100元。一场撕逼大战开始了。


#### thread.join()
在很多情况下，主线程创建并启动子线程，如果子线程中要进行大量的耗时运算，主线程将可能早于子线程结束。如果主线程需要知道子线程的执行结果时，就需要等待子线程执行结束了。主线程可以sleep(xx),但这样的xx时间不好确定，因为子线程的执行时间不确定，join()方法比较合适这个场景。
<img width="665" alt="image" src="https://github.com/BeggarLan/StudyNote/assets/49143666/71b9671b-e7fb-452a-b823-25147113c80b">
<img width="799" alt="image" src="https://github.com/BeggarLan/StudyNote/assets/49143666/6da35c25-81e4-41c8-aa8d-eda586e2e620">
<img width="676" alt="image" src="https://github.com/BeggarLan/StudyNote/assets/49143666/f64da8c0-1441-4134-a8a5-48e16eef4b81">
Join方法实现是通过wait（小提示：Object 提供的方法）。 当main线程调用t.join时候，main线程会获得线程对象t的锁（wait 意味着拿到该对象的锁),调用该对象的wait(等待时间)，直到该对象唤醒main线程 ，比如退出后。这就意味着main 线程调用t.join时，必须能够拿到线程t对象的锁。


#### Thread.interrupt方法
在线程wait、sleep等时，会抛出interruptedException异常 且 会把中断标记位重新置为false(方案：可以在catch内在调用intercept())
处于死锁状态时，不会理会中断

#### volatile
一写多读的场景适合用volatile

#### notifyAll
// notifyAll()写在这两个地方没有区别，notifyAll会通知wait的那些线程进就绪，等notifyAll所在的synchronized块执行完了后，wait的那些线程才会去执行
```java
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
```

#### 如何优雅的终止一个线程
对于 Java 而言，最正确的停止线程的方式是使用 interrupt。但 interrupt 仅仅起到通知被停止线程的作用。而对于被停止的线程而言，它拥有完全的自主权，它既可以选择立即停止，也可以选择一段时间后停止，也可以选择压根不停止。可能很多同学会疑惑，既然这样那这个存在的意义有什么尼，其实对于 Java 而言，期望程序之间是能够相互通知、协作的管理线程
* interrupt 停止线程
核心就是通过调用线程的 isInterrupt() 方法进而判断中断信号，当线程检测到为 true 时则说明接收到终止信号，此时我们需要做相应的处理。
```java
 Thread thread = new Thread(() -> {
            while (true) {
                //判断当前线程是否中断，
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("线程1 接收到中断信息，中断线程...中断标记：" + Thread.currentThread().isInterrupted());
                    //跳出循环，结束线程
                    break;
                }
                System.out.println(Thread.currentThread().getName() + "线程正在执行...");

            }
        }, "interrupt-1");
        //启动线程 1
        thread.start();

        //创建 interrupt-2 线程
        new Thread(() -> {
            int i = 0;
            while (i <20){
                System.out.println(Thread.currentThread().getName()+"线程正在执行...");
                if (i == 8){
                    System.out.println("设置线程中断...." );
                    //通知线程1 设置中断通知
                    thread.interrupt();

                }
                i ++;
                try {
                    TimeUnit.MILLISECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"interrupt-2").start();
```
当处于 sleep 或者wait时
```java
        //创建 interrupt-1 线程

        Thread thread = new Thread(() -> {
            while (true) {
                //判断当前线程是否中断，
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("线程1 接收到中断信息，中断线程...中断标记：" + Thread.currentThread().isInterrupted());
                    Thread.interrupted(); // //对线程进行复位，由 true 变成 false
                    System.out.println("经过 Thread.interrupted() 复位后，中断标记：" + Thread.currentThread().isInterrupted());

                    //再次判断是否中断，如果是则退出线程
                    if (Thread.currentThread().isInterrupted()) {
                        break;
                    }
                    break;
                }
                System.out.println(Thread.currentThread().getName() + "线程正在执行...");
                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "interrupt-1");
```
我们执行修改后的代码，发现如果 sleep、wait 等可以让线程进入阻塞的方法使线程休眠了，而处于休眠中的线程被中断，那么线程是可以感受到中断信号的，并且会抛出一个 InterruptedException 异常，同时清除中断信号，将中断标记位设置成 false。这样一来就不用担心长时间休眠中线程感受不到中断了，因为即便线程还在休眠，仍然能够响应中断通知，并抛出异常。

对于线程的停止，最优雅的方式就是通过 interrupt 的方式来实现，关于他的详细文章看之前文章即可，如 InterruptedException 时，再次中断设置，让程序能后续继续进行终止操作。不过对于 interrupt 实现线程的终止在实际开发中发现使用的并不是很多，很多都可能喜欢另一种方式，通过标记位。

* 用 volatile 标记位的停止方法，但是线程在wait或者sleep时就不适应了。
```java
public class MarkThreadTest {

    //定义标记为 使用 volatile 修饰
    private static volatile  boolean mark = false;

    @Test
    public void markTest(){
        new Thread(() -> {
            //判断标记位来确定是否继续进行
            while (!mark){
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程执行内容中...");
            }
        }).start();

        System.out.println("这是主线程走起...");
        try {
            TimeUnit.SECONDS.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //10秒后将标记为设置 true 对线程可见。用volatile 修饰
        mark = true;
        System.out.println("标记位修改为："+mark);
    }
}
```


## 信号量和管程
(信号量和管程)[https://www.cnblogs.com/binarylei/p/12544002.html]

## synchronized：https://juejin.cn/post/6888137809929093133

偏向锁为啥废弃，谈谈Java Synchronized 的锁机制，以及管程：https://cloud.tencent.com/developer/article/1759559
轻量级锁、锁升级：https://bbs.huaweicloud.com/blogs/335820

锁升级的图：https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/feec7b70b0f944c4aa19899d5d0c7db4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp

#### 锁优化
Synchronized 只在 JDK 1.6 以前性能才很差，因为这之前的 JVM 实现都是重量级锁，直接调用 ObjectMonitor 的 enter 和 exit。从 JDK 1.6 开始，HotSpot 虚拟机就增加了上述所说的几种优化：

偏向锁
轻量级锁
自旋锁
其余还有：

适应性自旋
锁消除
锁粗化

* 锁消除
这属于编译器对锁的优化，JIT 编译器在动态编译同步块时，会使用逃逸分析技术，判断同步块的锁对象是否只能被一个对象访问，没有发布到其它线程。
如果确认没有“逃逸”，JIT 编译器就不会生成 Synchronized 对应的锁申请和释放的机器码，就消除了锁的使用。

* 锁粗化
JIT 编译器动态编译时，如果发现几个相邻的同步块使用的是同一个锁实例，那么 JIT 编译器将会把这几个同步块合并为一个大的同步块，从而避免一个线程“反复申请、释放同一个锁“所带来的性能开销。

* 减小锁粒度
我们在代码实现时，尽量减少锁粒度，也能够优化锁竞争。

* 总结
其实现在 Synchronized 的性能并不差，偏向锁、轻量级锁并不会从用户态到内核态的切换；只有在竞争十分激烈的时候，才会升级到重量级锁。
Synchronized 的锁是由 JVM 实现的。
偏向锁在jdk15已经被废弃了。

## aqs
https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html
https://xie.infoq.cn/article/0223d5e5f19726b36b084b10d

比如线程加入等待队列的时候，需要把tail尾节点设置成当前节点，设置方法：
```java
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
```

#### java.util.concurrent中对aqs的应用场景：
* ReentrantLock	        使用AQS保存锁重复持有的次数。当一个线程获取锁时，ReentrantLock记录当前获得锁的线程标识，用于检测是否重复获取，以及错误线程试图解锁操作时异常情况的处理。
* Semaphore	            使用AQS同步状态来保存信号量的当前计数。tryRelease会增加计数，acquireShared会减少计数。
* CountDownLatch    	    使用AQS同步状态来表示计数。计数为0时，所有的Acquire操作（CountDownLatch的await方法）才可以通过。
* ReentrantReadWriteLock	使用AQS同步状态中的16位保存写锁持有的次数，剩下的16位用于保存读锁的持有次数。
* ThreadPoolExecutor    	Worker利用AQS同步状态实现对独占线程变量的设置（tryAcquire和tryRelease）。


## cpu密集型，线程池的线程数量一般是：cpu核数+1
加1：即使当计算密集型的线程偶尔由于缺页或者其他原因而暂停时，这个额外的线程也能确保cpu的时钟周期不会被浪费


## ThreadLocal
* 不存数据，类似于一个工具方法，真正存放数据的是Thread.threadLocalMap
```java
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

ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
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
```

#### 内存泄漏
ThreadLocal的原理和内存泄露分析：https://juejin.cn/post/7091649401629704223
内存泄漏源码分析https://www.jianshu.com/p/dde92ec37bd1

* ThreadLocalMap 中的 key 使用了弱引用，会导致 value 出现内存泄漏。
case： 假设在业务代码中使用完 ThreadLocal，threadLocalRef 被回收了
    由于 ThreadLocalMap 只持有 ThreadLocal 的弱引用，没有任何强引用指向 threadlocal 实例，所以 threadlocal 就可以顺利被 GC 回收，此时 Entry 中的 key = null
    在没有手动删除这个 Entry 以及 CurrentThread 依然运行的前提下，也存在有强引用链 threadRef -> currentThread -> threadLocalMap -> entry -> value，value 不会被回收，而这块 value 永远不会被访问到了，导致 value 内存泄漏
导致内存泄漏的原因
    1. 没有手动删除相应的 Entry 对象
    2. 当前线程依然在运行
* 如何解决内存泄露
    1. 使用完 ThreadLocal，调用其 remove 方法删除对应的 Entry
        ```java
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
        ```
    2. 使用完 ThreadLocal，当前 Thread 也随之运行结束（不好控制，线程池中的核心线程不会销毁）

#### 延伸：
threadLocalMap为啥不用weakHashMap呢，而是自己写了一个类:ThreadLocalMap；weakHashMap的entry也是弱引用，而且比threadLocal好的一点是，他对value也会释放
threadLocal释放value是有问题的，如果使用不当，很容易内存泄漏
    1. 线程退出(Thread.exit())中会让threadLocalMap=null：这里需要注意线程池的复用不会停止线程
    2. threadLocal.remove
    3. threadLocal的get和set(只有hash冲突的时候发现key为null了才会去让value=null)

我觉得的一个点是，threadLocalMap的hash方式是开放地址法，可以节省空间，但感觉这并不是主要原因


## 引用
#### Reference
<img width="1377" alt="image" src="https://github.com/BeggarLan/StudyNote/assets/49143666/1fc840ab-c0b0-4a59-acee-5c30c17e1ba7">

#### 引用队列
<img width="1548" alt="image" src="https://github.com/BeggarLan/StudyNote/assets/49143666/5c4483a1-4839-439e-a5ff-181ab6080bab">
* 当引用的对象被回收时，reference会加入到引用队列中
* 应用场景：WeakHashMap, 相对于ThreadLoacl的Entry的value的内存泄漏，WeakHashMap会查询引用队列去清空value
```java
class WeakHashMap {
    private final ReferenceQueue<Object> queue = new ReferenceQueue<>();
}
```
<img width="949" alt="image" src="https://github.com/BeggarLan/StudyNote/assets/49143666/faf9ac74-d0c9-47cc-a2b1-1250800dd221">

