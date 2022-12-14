[TOC]



# 介绍



#  运行过程

![image-20220916163311463](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916163311463.png)

编译后的字节码在没有经过JIT(实时编译器)编译前，是通过字节码解释器进行解释执行。其执行原理为：字节码解释器读取内存中的字节码，按照顺序读取字节码指令，读取一个指令就将其翻译成固定的操作，根据这些操作进行分支，循环，跳转等动作。

## **JIT优化**

JVM先将字节码翻译为对应的机器指令，然后执行机器指令。很显然，这样经过解释执行，其执行速度必然不如直接执行二进制字节码文件。

而为了提高执行速度，便引入了 JIT 技术。当JVM发现某个方法或代码块运行特别频繁的时候，就会认为这是“热点代码”（Hot Spot Code)。然后JIT会把部分“热点代码”编译成本地机器相关的机器码，并进行优化，然后再把编译后的机器码缓存起来，以备下次使用。

## **Hot Spot编译**

当 JVM 执行代码时，它并不是立即开始编译代码的。这主要有两个原因：

首先，如果这段代码本身在将来只会被执行一次，那么从本质上看，编译就是在浪费精力。因为将代码翻译成 java 字节码相对于编译这段代码并执行代码来说，要快很多。当然，如果一段代码频繁的调用方法，或是一个循环，也就是这段代码被多次执行，那么编译就非常值得了。

想要触发JIT编译，首先要识别出热点代码。目前主要的热点代码识别方式是热点探测(Hot Spot Detection)，有以下两种：

1. **基于采样方式探测（Sample Based Hot Spot Detection）**：周期性检测各个线程的栈顶，发现某个方法经常出现在栈顶，就认为是热点方法。好处就是简单，缺点就是无法精确确认一个方法的热度。容易受线程阻塞或别的原因干扰热点探测。
2. **基于计数器的热点探测（Counter Based Hot Spot Detection）**：采用这种方法的虚拟机会为每个方法，甚至是代码块建立计数器，统计方法的执行次数，某个方法超过阀值就认为是热点方法，触发JIT编译。

在HotSpot虚拟机中使用的是第二种——基于计数器的热点探测方法，因此它为每个方法准备了两个计数器：**方法调用计数器(记录一个方法被调用次数)和回边计数器(循环的运行次数)**。

## 编译阈值

当 JVM 执行一个 Java 方法，它会检查方法调用计数器和回边计数器的总和，以决定这个方法是否有资格被编译。如果有，则这个方法将排队等待编译。这种编译形式并没有一个官方的名字，但是一般被叫做标准编译。

这种编译是一个异步的过程，它允许程序在代码正在编译时被继续执行。

但是如果方法里有一个很长的循环或者是一个永远都不会退出并提供了所有逻辑的程序会怎么样呢？这种情况下，JVM 需要编译循环而并不等待方法被调用。所以每执行完一次循环，分支计数器都会自增和自检。如果分支计数器计数超出其自身阈值，那么这个循环（并不是整个方法）将具有被编译资格。

这种编译叫做**栈上替换（OSR）**，因为即使循环被编译了，这也是不够的：JVM 必须有能力当循环正在运行时，开始执行此循环已被编译的版本。换句话说，**如果一个循环被栈上替换方式所编译，那么下一次循环迭代则会执行新编译的代码**。

## 编译优化

JIT除了具有缓存的功能外，还会对代码做各种优化，包括：逃逸分析、 锁消除、 锁膨胀、 方法内联、 空值检查消除、 类型检测消除、 公共子表达式消除



## 类加载



# 内存空间

## JVM内存模型

![image-20220916164046134](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916164046134.png)

1、**程序计数器**
**程序计数器（Program CounterRegister）**是一块较小的内存空间，它的作用可以看做是当前线程所执行的字节码的行号指示器。此内存区域是唯一一个在Java 虚拟机规范中没有规定任何 OutOfMemoryError 情况的区域。

2、**Java 虚拟机栈**
**每个方法被执行的时候都会同时创建一个栈帧(Stack Frame）**用于存储**局部变量表、操作栈、动态链接、方法出口等信息**。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

3、**本地方法栈**
本地方法栈（Native MethodStacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java 方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native 方法服务。



### JVM堆

![image-20220916164218291](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916164218291.png)

- **永久代（Perm）**【过时】：主要保存class，method，field等对象，该空间大小，取决于系统启动加载类的数量，一般该区域内存溢出均是启动时溢出。java.lang.OutOfMemoryError: PermGen space
- **老年代（Old）**：一般是经过多次垃圾回收（GC）没有被回收掉的对象
- **新生代（Eden）**：新创建的对象，
- **新生代（Survivor0）**：经过垃圾回收（GC）后，没有被回收掉的对象
- **新生代（Survivor1）**：同Survivor0相同，大小空间也相同，同一时刻Survivor0和Survivor1只有一个在用，一个为空

![image-20220916165725173](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916165725173.png)

默认新生代与老年代1:2，可以通过–XX:NewRatio 来指定。



### GC

三种垃圾收集算法：复制算法，标记-清除算法、标记-整理算法

1. **标记-清除算法**：首先标记出所有需要回收的对象，标记完成后统一回收所有被标记的对象。缺点：标记和清除两个过程效率都不高；标记清楚后会产生空间碎片，空间碎片导致分配较大对象时可能提前出发垃圾回收。
2. **复制算法**: 将可用内存分为两个区域，每次只使用其中一块，当使用的那一块内存用完时，将还存活的对象复制到另外一块内存中，然后把已使用过的内存空间一次清理掉。优点：解决的空间碎片问题，实现简单。缺点：将内存缩小为两块，内存使用率不高。复制操作频繁效率变低。
3. **标记-整理算法**：可回收对象标记后，让所有存活的对象向一端移动，然后清理掉边界以外的内存。优点：不会产生空间碎片，比复制算法提高了内存空间利用率。

复制算法用在年轻代的垃圾回收中，标记整理和标记清除算法用在老年代垃圾回收的收集器中。

#### 1. Serial收集器

串行收集器是最古老，最稳定以及效率高的收集器
可能会产生较长的停顿，只使用一个线程去回收

![image-20220916170221963](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916170221963.png)

#### 2. 并行收集器

##### ParNew

Serial收集器新生代的并行版本
在新生代回收时使用复制算法
多线程，需要多核支持

![image-20220916170246094](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916170246094.png)

##### Parallel收集器

类似ParNew 
新生代复制算法 
老年代标记-压缩 
更加关注吞吐量 

![image-20220916170326719](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916170326719.png)

#### 3. CMS收集器

CMS运行过程比较复杂，着重实现了标记的过程，可分为

1. 初始标记（会产生全局停顿）

根可以直接关联到的对象
速度快
2. 并发标记（和用户线程一起） 

主要标记过程，标记全部对象
3. 重新标记 （会产生全局停顿） 

由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正
4. 并发清除（和用户线程一起） 

基于标记结果，直接清理对象
![image-20220916170422174](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916170422174.png)

这里就能很明显的看出，为什么CMS要使用标记清除而不是标记压缩，如果使用标记压缩，需要多对象的内存位置进行改变，这样程序就很难继续执行。但是标记清除会产生大量内存碎片，不利于内存分配。 

#### 4. G1收集器

1. 标记阶段，首先初始标记(Initial-Mark),这个阶段是停顿的(Stop the World Event)，并且会触发一次普通Mintor GC。对应GC log:GC pause (young) (inital-mark)

2. Root Region Scanning，程序运行过程中会回收survivor区(存活到老年代)，这一过程必须在young GC之前完成。

3. Concurrent Marking，在整个堆中进行并发标记(和应用程序并发执行)，**此过程可能被young GC中断**。在并发标记阶段，若发现区域对象中的所有对象都是垃圾，那个这个区域会被立即回收(图中打X)。同时，并发标记过程中，会计算每个区域的对象活性(区域中存活对象的比例)。

   ![image-20220916170639898](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916170639898.png)

4. Remark, 再标记，会有短暂停顿(STW)。再标记阶段是用来收集 并发标记阶段 产生新的垃圾(并发阶段和应用程序一同运行)；G1中采用了比CMS更快的初始快照算法:snapshot-at-the-beginning (SATB)。

5. Copy/Clean up，多线程清除失活对象，会有STW。G1将回收区域的存活对象拷贝到新区域，清除Remember Sets，并发清空回收区域并把它返回到空闲区域链表中。

   ![image-20220916170717986](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916170717986.png)

6. 复制/清除过程后。回收区域的活性对象已经被集中回收到深蓝色和深绿色区域。



### 堆与方法区（元空间）

![image-20220916164514922](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916164514922.png)

**Java7及以前版本的Hotspot中方法区位于永久代中**。同时，永久代和堆是相互隔离的，但它们使用的物理内存是连续的。

永久代的垃圾收集是和老年代捆绑在一起的，因此无论谁满了，都会触发永久代和老年代的垃圾收集。

但在Java7中永久代中存储的部分数据已经开始转移到Java Heap或Native Memory中了。比如，符号引用(Symbols)转移到了Native Memory；字符串常量池(interned strings)转移到了Java Heap；类的静态变量(class statics)转移到了Java Heap。

然后，在Java8中，时代变了，Hotspot取消了永久代。永久代真的成了永久的记忆。永久代的参数-XX:PermSize和-XX：MaxPermSize也随之失效。

#### 元空间

在Java8中，元空间(Metaspace)登上舞台，方法区存在于元空间(Metaspace)。同时，元空间不再与堆连续，而且是存在于本地内存（Native memory）。

![image-20220916164643918](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916164643918.png)

元空间存在于本地内存，意味着只要本地内存足够，它不会出现像永久代中“java.lang.OutOfMemoryError: PermGen space”这种错误。默认情况下元空间是可以无限使用本地内存的，但为了不让它如此膨胀，JVM同样提供了参数来限制它使用的使用。

- -XX:MetaspaceSize，class metadata的初始空间配额，以bytes为单位，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当的降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize（如果设置了的话），适当的提高该值。
- -XX：MaxMetaspaceSize，可以为class metadata分配的最大空间。默认是没有限制的。
- -XX：MinMetaspaceFreeRatio,在GC之后，最小的Metaspace剩余空间容量的百分比，减少为class metadata分配空间导致的垃圾收集。
- -XX:MaxMetaspaceFreeRatio,在GC之后，最大的Metaspace剩余空间容量的百分比，减少为class metadata释放空间导致的垃圾收集。





## Java 内存模型

### CPU底层

我们甚至可以把 **内存可以看作外存的高速缓存**，程序运行的时候我们把外存的数据复制到内存，由于内存的处理速度远远高于外存，这样提高了处理速度。

总结：**CPU Cache 缓存的是内存数据用于解决 CPU 处理速度和内存不匹配的问题，内存缓存的是硬盘数据用于解决硬盘访问速度过慢的问题。**

![image-20220916165159836](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916165159836.png)

**CPU Cache 的工作方式：** 先复制一份数据到 CPU Cache 中，当 CPU 需要用到的时候就可以直接从 CPU Cache 中读取数据，当运算完成后，再将运算得到的数据写回 Main Memory 中。但是，这样存在 **内存缓存不一致性的问题** ！

**CPU 为了解决内存缓存不一致性问题可以通过制定缓存一致协议（比如 [MESI 协议open in new window](https://zh.wikipedia.org/wiki/MESI协议)）或者其他手段来解决。** 这个缓存缓存一致性协议指的是在 CPU 高速缓存与主内存交互的时候需要准守的原则和规范。不同的 CPU 中，使用的缓存一致性协议通常也会有所不同。

![image-20220916165238469](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916165238469.png)

### 指令重排序

为了提升执行速度/性能，计算机在执行程序代码的时候，会对指令进行重排序。

常见的指令重排序有下面 2 种情况：

- **编译器优化重排** ：编译器（包括 JVM、JIT 编译器等）在不改变单线程程序语义的前提下，重新安排语句的执行顺序。
- **指令并行重排** ：现代处理器采用了指令级并行技术(Instruction-Level Parallelism，ILP)来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

**指令重排序可以保证串行语义一致，但是没有义务保证多线程间的语义也一致** ，所以在多线程下，指令重排序可能会导致一些问题。

编译器和处理器的指令重排序的处理方式不一样。对于编译器，通过**禁止特定类型的编译器的当时来禁止重排序**。对于处理器，通过插入**内存屏障（Memory Barrier，或有时叫做内存栅栏，Memory Fence）的方式**来禁止特定类型的处理器重排序。指令并行重排和内存系统重排都属于是处理器级别的指令重排序。

### JMM(Java Memory Model)

Java 内存模型（JMM） 抽象了线程和主内存之间的关系，就比如说线程之间的共享变量必须存储在主内存中。

在 JDK1.2 之前，Java 的内存模型实现总是从 **主存** （即共享内存）读取变量，是不需要进行特别的注意的。而在当前的 Java 内存模型下，线程可以把变量保存 **本地内存** （比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成数据的不一致。

**什么是主内存？什么是本地内存？**

- **主内存** ：所有线程创建的实例对象都存放在主内存中，不管该实例对象是成员变量还是方法中的本地变量(也称局部变量)
- **本地内存** ：每个线程都有一个私有的本地内存来存储共享变量的副本，并且，每个线程只能访问自己的本地内存，无法访问其他线程的本地内存。本地内存是 JMM 抽象出来的一个概念，存储了主内存中的共享变量副本。

![image-20220916165529849](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916165529849.png)



# 多线程

## 锁

![image-20220916165034043](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916165034043.png)

### synchronized

synchronized 通过当前线程持有对象锁，从而拥有访问权限，而其他没有持有当前对象锁的线程无法拥有访问权限，保证在同一时刻，只有一个线程可以执行某个方法或者某个代码块，从而保证线程安全。synchronized 可以保证线程的可见性，**synchronized 属于隐式锁，锁的持有与释放都是隐式的，我们无需干预**。synchronized最主要的三种应用方式：

- 修饰实例方法：作用于当前实例加锁，进入同步代码前要获得当前实例的锁
- 修饰静态方法：作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁
- 修饰代码块：指定加锁对象，进入同步代码库前要获得给定对象的锁

#### 原理

synchronized 锁机制在 Java 虚拟机中的同步是**基于进入和退出监视器锁对象** monitor 实现的。每个对象的对象头都关联着一个 monitor 对象，当一个 monitor 被某个线程持有后，它便处于锁定状态。

![image-20220916171117342](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916171117342.png)

在 HotSpot 虚拟机中，monitor 是由 ObjectMonitor 实现的，每个等待锁的线程都会被封装成 ObjectWaiter 对象，ObjectMonitor 中有两个集合，**WaitSet 和 EntryList，用来保存 ObjectWaiter 对象列表** ，owner 区域指向持有 ObjectMonitor 对象的线程。

当多个线程同时访问一段同步代码时，首先会进入 _EntryList 集合尝试获取 moniter，当线程获取到对象的 monitor 后进入 _Owner 区域并把 _owner 变量设置为当前线程，同时 monitor 中的计数器 count 加1；若线程调用 wait() 方法，将释放当前持有的 monitor，count自减1，owner 变量恢复为 null，同时该线程进入 _WaitSet 集合中等待被唤醒。若当前线程执行完毕也将释放 monitor 并复位变量的值，以便其他线程获取 monitor。如下图所示：

#### 优化

在早期版本中，synchronized 属于重量级锁，效率低下，因为监视器锁 monitor 是**依赖于操作系统的 Mutex 互斥量**来实现的，操作系统实现线程之间的切换时**需要从用户态转换到核心态**，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。在 JDK6 之后，synchronized 在 JVM 层面做了优化，减少锁的获取和释放所带来的性能消耗，主要优化方向有以下几点：

锁升级：**偏向锁->轻量级锁->自旋锁->重量级锁**

锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁，但是锁的升级是单向的，**只能从低到高升级，不会出现锁的降级**。**重量级锁基于从操作系统的互斥量**实现的，而**偏向锁与轻量级锁**不同，他们是通过 **CAS 并配合 Mark Word** 一起实现的。



##### synchronized 的 Mark word 标志位：

synchronized 使用的锁对象是存储在 Java 对象头里的，那么 Java 对象头是什么呢？对象实例分为：

- 对象头
  - Mark Word
  - 指向类的指针
  - 数组长度
- 实例数据
- 对齐填充

其中，Mark Word 记录了对象的 hashcode、分代年龄、锁标记位相关的信息，由于对象头的信息是与对象自身定义的数据没有关系的额外存储成本，因此考虑到 JVM 的空间效率，Mark Word 被设计成为一个非固定的数据结构，以便存储更多有效的数据，它会根据对象本身的状态复用自己的存储空间，在 32位 JVM 中的长度是 32 位

![image-20220916171732311](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916171732311.png)

##### 升级机制

（1）**偏向锁**：如果一个线程获得了锁，那么进入偏向模式，当这个线程再次请求锁的时候，只需去对象头的 Mark Word 中判断偏向线程ID是否指向它自己，无需再进入 monitor 中去竞争对象，这样就省去了大量锁申请的操作，适用于连续多次都是同一个线程申请相同的锁的场景。偏向锁只有初始化的时候需要一次 CAS 操作，但如果出现其他线程竞争锁资源，那么偏向锁就会被撤销，并升级为轻量级锁。

（2）**轻量级锁**：不需要申请互斥量，允许短时间内的锁竞争，**每次申请、释放锁都至少需要一次 CAS**，适用于多个线程交替执行同步代码块的场景

（3）**自旋锁**：自旋锁假设在不久将来，当前的线程可以获得锁，因此在轻量级锁升级成为重量级锁之前，**虚拟机会让当前想要获取锁的线程做几个空循环，在经过若干次循环后，如果得到锁，就顺利进入临界区，如果还不能获得锁，那就会将线程在操作系统层面挂起**。

> 这种方式确实可以提升效率的，但是当线程越来越多竞争很激烈时，占用 CPU 的时间变长会导致性能急剧下降，因此 JVM 对于自旋锁有一定的次数限制，可能是50或者100次循环后就放弃，直接挂起线程，让出CPU资源。

（4）**自适应自旋锁**：自适应自旋解决的是 “锁竞争时间不确定” 的问题，**自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定**。

- 如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如100个循环。
- 相反的，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能减少自旋时间甚至省略自旋过程，以避免浪费处理器资源。

> 但自旋锁带来的副作用就是不公平的锁机制：处于阻塞状态的线程，并没有办法立刻竞争被释放的锁。然而，处于自旋状态的线程，则很有可能优先获得这把锁。

（5）**重量级锁**：适用于多个线程同时执行同步代码块的场景，且锁竞争时间长。在这个状态下，未抢到锁的线程都会进入到 Monitor 中并阻塞在 _WaitSet 集合中。



### ReentrantLock

ReentrantLock底层使用了CAS+AQS队列实现。

AQS使用一个FIFO的队列（也叫CLH队列，是[CLH锁](https://link.zhihu.com/?target=https%3A//blog.csdn.net/claram/article/details/83828768)的一种变形），表示排队**等待锁的线程**。队列**头节点**称作“哨兵节点”或者“哑节点”，它不与任何线程关联。**其他的节点**与等待线程关联，每个节点维护一个等待状态waitStatus。结构如下图所示：

![image-20220916172719895](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/image-20220916172719895.png)

流程：

1. ReentrantLock先通过CAS尝试获取锁，

2. 1. 如果此时锁已经被占用，该线程加入AQS队列并wait()

   2. 当前驱线程的锁被释放，挂在CLH队列为首的线程就会被notify()，然后继续CAS尝试获取锁，此时：

   3. 1. 非公平锁，如果有其他线程尝试lock()，有可能被其他刚好申请锁的线程**抢占**。
      2. 公平锁，只有在CLH**队列头的线程**才可以获取锁，**新来的线程**只能插入到队尾。

#### lock() 和 unlock() 的实现

##### lock()函数

如果成功通过CAS修改了state，指定当前线程为该锁的独占线程，标志自己成功获取锁。

如果CAS失败的话，调用acquire();

```java
final void lock() { //非公平锁
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

final void lock() { //公平锁
    acquire(1);
}
```



## 实现多线程

1. 继承Thread类，重写run()方法
2. 实现Runnable接口，重写run()
3. 带返回值的线程(实现implements  Callable<返回值类型>)



## 线程池

`Executor` 框架是 Java5 之后引进的，在 Java 5 之后，通过 `Executor` 来启动线程比使用 `Thread` 的 `start` 方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免 this 逃逸问题。

> 补充：this 逃逸是指在构造函数返回之前其他线程就持有该对象的引用. 调用尚未构造完全的对象的方法可能引发令人疑惑的错误。

`Executor` 框架不仅包括了线程池的管理，还提供了线程工厂、队列以及拒绝策略等，`Executor` 框架让并发编程变得更加简单。

### 组成成分

#### 任务(`Runnable` /`Callable`)

执行任务需要实现的 **`Runnable` 接口** 或 **`Callable`接口**。**`Runnable` 接口**或 **`Callable` 接口** 实现类都可以被 **`ThreadPoolExecutor`** 或 **`ScheduledThreadPoolExecutor`** 执行。

#### 任务的执行(`Executor`)

如下图所示，包括任务执行机制的核心接口 **`Executor`** ，以及继承自 `Executor` 接口的 **`ExecutorService` 接口。`ThreadPoolExecutor`** 和 **`ScheduledThreadPoolExecutor`** 这两个关键类实现了 **ExecutorService 接口**。

**这里提了很多底层的类关系，但是，实际上我们需要更多关注的是 `ThreadPoolExecutor` 这个类，这个类在我们实际使用线程池的过程中，使用频率还是非常高的。**

> **注意：** 通过查看 `ScheduledThreadPoolExecutor` 源代码我们发现 `ScheduledThreadPoolExecutor` 实际上是继承了 `ThreadPoolExecutor` 并实现了 ScheduledExecutorService ，而 `ScheduledExecutorService` 又实现了 `ExecutorService`，正如我们下面给出的类关系图显示的一样。

**`ThreadPoolExecutor` 类描述:**

```java
//AbstractExecutorService实现了ExecutorService接口
public class ThreadPoolExecutor extends AbstractExecutorService
```

**`ScheduledThreadPoolExecutor` 类描述:**

```java
//ScheduledExecutorService继承ExecutorService接口
public class ScheduledThreadPoolExecutor
        extends ThreadPoolExecutor
        implements ScheduledExecutorService
```

![任务的执行相关接口](D:/project/Interview/%E8%AF%AD%E8%A8%80/image/Java/%E4%BB%BB%E5%8A%A1%E7%9A%84%E6%89%A7%E8%A1%8C%E7%9B%B8%E5%85%B3%E6%8E%A5%E5%8F%A3.27457eb8.png)

#### 异步计算的结果(`Future`)

**`Future`** 接口以及 `Future` 接口的实现类 **`FutureTask`** 类都可以代表异步计算的结果。

当我们把 **`Runnable`接口** 或 **`Callable` 接口** 的实现类提交给 **`ThreadPoolExecutor`** 或 **`ScheduledThreadPoolExecutor`** 执行。（调用 `submit()` 方法时会返回一个 **`FutureTask`** 对象）



#### ThreadPoolExecutor 类分析

```java
    /**
     * 用给定的初始参数创建一个新的ThreadPoolExecutor。
     */
    public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                              int maximumPoolSize,//线程池的最大线程数
                              long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
                              ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                              RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                               ) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

- **`corePoolSize` :** 核心线程数线程数定义了最小可以同时运行的线程数量。
- **`maximumPoolSize` :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- **`workQueue`:** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

其余参数：

1. **`keepAliveTime`**:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
2. **`unit`** : `keepAliveTime` 参数的时间单位。
3. **`threadFactory`** :executor 创建新线程的时候会用到。
4. **`handler`** :饱和策略。关于饱和策略下面单独介绍一下。



#### 饱和策略

**`ThreadPoolExecutor` 饱和策略定义:**

如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了任务时，`ThreadPoolTaskExecutor` 定义一些策略:

- **`ThreadPoolExecutor.AbortPolicy`** ：抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`** ：调用执行自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
- **`ThreadPoolExecutor.DiscardPolicy`** ：不处理新任务，直接丢弃掉。
- **`ThreadPoolExecutor.DiscardOldestPolicy`** ： 此策略将丢弃最早的未处理的任务请求。
