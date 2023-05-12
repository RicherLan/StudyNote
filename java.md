## JVM内存
#### 内存区域
* 运行时数据区域 + 直接内存(堆外内存)
<img width="890" alt="image" src="https://user-images.githubusercontent.com/49143666/236633385-e36ba90f-298a-408c-ac28-43e480edcc87.png">

* 虚拟机栈
<img width="876" alt="image" src="https://user-images.githubusercontent.com/49143666/236751529-ab2ed75f-7b71-4feb-8b1c-216f87a7ed9d.png">
<img width="716" alt="image" src="https://user-images.githubusercontent.com/49143666/236751658-28f454a0-a8cf-4397-95fb-9c79948fa709.png">
<img width="872" alt="image" src="https://user-images.githubusercontent.com/49143666/236753443-6901e029-89bb-4345-ac52-2285ef6b3d80.png">

* 本地方法栈
<img width="711" alt="image" src="https://user-images.githubusercontent.com/49143666/236756614-e66c1086-a678-4889-8990-3d8d643c2cc9.png">
比如hashCode()方法，就是非java语言实现的，和虚拟机栈类似，在hotspot虚拟机是不区分虚拟机栈和本地方法栈的(合二为一了，JVM只是规范，具体的实现不同)

* 程序计数器
注意，记录的是虚拟机栈的，本地方法栈的不记录

* 方法区
类加载的时候，class、静态变量、static块等都会放在方法区

* 堆

##### 常量池


#### JVM对象的分配
* 对象的内存布局
<img width="880" alt="image" src="https://user-images.githubusercontent.com/49143666/236774988-e6774555-03dd-4d8a-887d-9ba6d386f4cc.png">

* 对象的创建过程
<img width="907" alt="image" src="https://user-images.githubusercontent.com/49143666/236771787-796703d2-4517-46db-8ee2-7d755575522d.png">


#### JVM垃圾回收
* 对象存活判定
gcRoots不止这4种(比如还有class对象等)，常见的就是这4种
<img width="771" alt="image" src="https://user-images.githubusercontent.com/49143666/236790495-cd241fd1-4861-44c3-a685-3e261d9290b0.png">
* class对象回收的条件
<img width="523" alt="image" src="https://user-images.githubusercontent.com/49143666/236794195-3cca7de0-f336-4631-ad0a-f5098661f7af.png">

* 引用
虚引用例子
<img width="740" alt="image" src="https://user-images.githubusercontent.com/49143666/236800412-592e4ab7-9672-4eba-8d54-2a4f7553d918.png">
<img width="680" alt="image" src="https://user-images.githubusercontent.com/49143666/236800477-9627d956-4220-4d3b-9cb4-4d621d9bc73e.png">

* 对象的分配策略
<img width="897" alt="image" src="https://user-images.githubusercontent.com/49143666/237001162-8cacc321-8012-4105-bc28-a8f903c14b57.png">

* JIT即时编译和对象逃逸分析
所以并不是所有的对象都在堆上分配(几乎所有的对象都在堆上分配)
<img width="678" alt="image" src="https://user-images.githubusercontent.com/49143666/237003117-ebe95441-39b7-4256-96c9-fa5e8dd52fed.png">

* 垃圾回收
<img width="784" alt="image" src="https://user-images.githubusercontent.com/49143666/237006577-dae301c2-f5ea-4eed-986d-a3d74854521d.png">

###### 垃圾回收算法
* 复制算法：新生代适合，新生代大部分都是垃圾
<img width="816" alt="image" src="https://user-images.githubusercontent.com/49143666/237010390-b895ea02-9756-494a-8881-77c4078227e1.png">
* 优化版的复制算法：垃圾占90%，不过有的垃圾回收器是动态调整各区域大小的(比如Parallel Scavenge回收器)
空间利用率只有一半，因此引入了s0和s1去(from和to)，Eden：from：to = 8:1:1，空间利用率为90%。对象在eden区中分配，存活对象会去交换区中的一个
<img width="905" alt="image" src="https://user-images.githubusercontent.com/49143666/237011426-fff21b1c-1be2-47f9-add6-ac9a4fc049bc.png">

* 标记清除算法 ：老年代适合，老年代大部分不是垃圾
<img width="807" alt="image" src="https://user-images.githubusercontent.com/49143666/237012789-d7405a04-ce53-4c21-8576-dbbb00939145.png">

* 标记整理算法：老年代适合，老年代大部分不是垃圾； 先标记，在整理，在清除：整理完了可以一下子整批清除(所以不能先清除在整理)
<img width="785" alt="image" src="https://user-images.githubusercontent.com/49143666/237032266-65f6e30f-5e93-4403-b246-8b0b7b434d24.png">

###### 垃圾回收器
* 常见的垃圾回收器
<img width="924" alt="image" src="https://user-images.githubusercontent.com/49143666/237013980-d4e59154-63d3-4749-bd29-77cd428441b9.png">

* CMS垃圾回收器：以最短的暂停时间(STW)为优先的
<img width="900" alt="image" src="https://user-images.githubusercontent.com/49143666/237031772-ec395621-3000-4bfd-84ef-4c7969acb832.png">
<img width="822" alt="image" src="https://user-images.githubusercontent.com/49143666/237031807-4ce617e0-3504-45ca-b922-34838e82b35a.png">


## HashMap
(jdk1.8HashMap)[https://tech.meituan.com/2016/06/24/java-hashmap.html]

#### 重要结构、字段
* Node
```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node

        Node(int hash, K key, V value, Node<K,V> next) { ... }
        public final K getKey(){ ... }
        public final V getValue() { ... }
        public final String toString() { ... }
        public final int hashCode() { ... }
        public final V setValue(V newValue) { ... }
        public final boolean equals(Object o) { ... }
}
```
* transient Node<K,V>[] table;
* int threshold;             // 所能容纳的key-value对极限 
* final float loadFactor;    // 负载因子
* transient int modCount;
* transient int size;  

#### 要点
* HashMap就是使用哈希表来存储的。哈希表为解决冲突，可以采用开放地址法和链地址法等来解决问题，Java中HashMap采用了链地址法。链地址法，简单来说，就是数组加链表的结合。在每个数组元素上都一个链表结构，当数据被Hash后，得到数组下标，把数据放在对应下标元素的链表上
* Hash算法计算结果越分散均匀，Hash碰撞的概率就越小，map的存取效率就会越高。如果哈希桶数组很大，即使较差的Hash算法也会比较分散，如果哈希桶数组数组很小，即使好的Hash算法也会出现较多碰撞，所以就需要在空间成本和时间成本之间权衡，其实就是在根据实际情况确定哈希桶数组的大小，并在此基础上设计好的hash算法减少Hash碰撞。那么通过什么方式来控制map使得Hash碰撞的概率又小，哈希桶数组（Node[] table）占用空间又少呢？答案就是好的Hash算法和扩容机制
* 首先，Node[] table的初始化长度length(默认值是16)，Load factor为负载因子(默认值是0.75)，threshold是HashMap所能容纳的最大数据量的Node(键值对)个数。threshold = length * Load factor。也就是说，在数组定义好长度之后，负载因子越大，所能容纳的键值对个数越多。

结合负载因子的定义公式可知，threshold就是在此Load factor和length(数组长度)对应下允许的最大元素数目，超过这个数目就重新resize(扩容)，扩容后的HashMap容量是之前容量的两倍。默认的负载因子0.75是对空间和时间效率的一个平衡选择，建议大家不要修改，除非在时间和空间比较特殊的情况下，如果内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值；相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1。
size这个字段其实很好理解，就是HashMap中实际存在的键值对数量。注意和table的长度length、容纳最大键值对数量threshold的区别。而modCount字段主要用来记录HashMap内部结构发生变化的次数，主要用于迭代的快速失败。强调一点，内部结构发生变化指的是结构发生变化，例如put新键值对，但是某个key对应的value值被覆盖不属于结构变化。
在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数，具体证明可以参考这篇文章，Hashtable初始化桶大小为11，就是桶大小设计为素数的应用（Hashtable扩容后不能保证还是素数）。HashMap采用这种非常规设计，主要是为了在取模和扩容时做优化，同时为了减少冲突，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。
这里存在一个问题，即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，则会严重影响HashMap的性能。于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能，其中会用到红黑树的插入、删除、查找等算法

#### hash计算方式
```java
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
     return h & (length-1);  //第三步 取模运算
}
```
* 为什么(h = key.hashCode()) ^ (h >>> 16)
1. 由于和（length-1）运算，length 绝大多数情况小于2的16次方。所以始终是hashcode 的低16位（甚至更低）参与运算。要是高16位也参与运算，会让得到的下标更加散列。
2. &和|都会使得结果偏向0或者1 ,并不是均匀的概念,所以用^

#### put方法（jdk1.8）
```java
 public V put(K key, V value) {
     // 对key的hashCode()做hash
     return putVal(hash(key), key, value, false, true);
 }
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 步骤①：tab为空则创建
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 步骤②：计算index，桶为空的话那么直接创建桶 
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 步骤③：桶节点key存在，直接覆盖value
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 步骤④：判断该链为红黑树，那么插入红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 步骤⑤：该链为链表，那么添加元素，如果超出链表数量限制那么转红黑树
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //链表长度大于8转换为红黑树进行处理
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                     // key已经存在直接覆盖value
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 步骤⑥：超过最大容量 就扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
}
```

#### 扩容（JDK1.7，鉴于JDK1.8融入了红黑树，较复杂，为了便于理解我们仍然使用JDK1.7的代码，好理解一些，本质上区别不大）
```java
// newTable[i]的引用赋给了e.next，也就是使用了单链表的头插入方式，同一位置上新元素总会被放在链表的头部位置；这样先放在一个索引上的元素终会被放到Entry链的尾部(如果发生了hash冲突的话），这一点和Jdk1.8有区别，下文详解。在旧数组中同一条Entry链上的元素，通过重新计算索引位置后，有可能被放到了新数组的不同位置上。
1 void resize(int newCapacity) {   //传入新的容量
 2     Entry[] oldTable = table;    //引用扩容前的Entry数组
 3     int oldCapacity = oldTable.length;         
 4     if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容前的数组大小如果已经达到最大(2^30)了
 5         threshold = Integer.MAX_VALUE; //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
 6         return;
 7     }
 8  
 9     Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
10     transfer(newTable);                         //！！将数据转移到新的Entry数组里
11     table = newTable;                           //HashMap的table属性引用新的Entry数组
12     threshold = (int)(newCapacity * loadFactor);//修改阈值
13 }

 1 void transfer(Entry[] newTable) {
 2     Entry[] src = table;                   //src引用了旧的Entry数组
 3     int newCapacity = newTable.length;
 4     for (int j = 0; j < src.length; j++) { //遍历旧的Entry数组
 5         Entry<K,V> e = src[j];             //取得旧Entry数组的每个元素
 6         if (e != null) {
 7             src[j] = null;//释放旧Entry数组的对象引用（for循环后，旧的Entry数组不再引用任何对象）
 8             do {
 9                 Entry<K,V> next = e.next;
10                 int i = indexFor(e.hash, newCapacity); //！！重新计算每个元素在数组中的位置
11                 e.next = newTable[i]; //标记[1]
12                 newTable[i] = e;      //将元素放在数组上
13                 e = next;             //访问下一个Entry链上的元素
14             } while (e != null);
15         }
16     }
17 }
```


* JDK1.8做了哪些优化。
经过观测可以发现，我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果
![image](https://github.com/BeggarLan/StudyNote/assets/49143666/486d4095-d618-4624-9ab0-69092bc4dd46)
元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：
![image](https://github.com/BeggarLan/StudyNote/assets/49143666/a3104081-4aa5-4c87-8642-caebb09fe5b5)
因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图：
![image](https://github.com/BeggarLan/StudyNote/assets/49143666/0397888f-6b16-45f9-b15b-dccdff893c2b)
这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。这一块就是JDK1.8新增的优化点。有一点注意区别，JDK1.7中rehash的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置，但是从上图可以看出，JDK1.8不会倒置

#### get方法（jdk1.8）
```java
 public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
}
```

## ConcurrentHashMap



