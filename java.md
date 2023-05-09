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



