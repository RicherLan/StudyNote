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


#### JVM对象的分配
* 对象的内存布局
<img width="880" alt="image" src="https://user-images.githubusercontent.com/49143666/236774988-e6774555-03dd-4d8a-887d-9ba6d386f4cc.png">

* 对象的创建过程
<img width="907" alt="image" src="https://user-images.githubusercontent.com/49143666/236771787-796703d2-4517-46db-8ee2-7d755575522d.png">


#### JVM垃圾回收
* 对象存活判定
<img width="771" alt="image" src="https://user-images.githubusercontent.com/49143666/236790495-cd241fd1-4861-44c3-a685-3e261d9290b0.png">


* 对象自我拯救

* 
