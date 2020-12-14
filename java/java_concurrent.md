# java并发知识点

## 简述线程、进程、程序的基本概念？

程序是含有指令和数据的文件，被储存在磁盘或者其他数据存储设备中，也就是说程序是静态的代码。

进程，进程是系统运行的基本单元，进程是动态的，可以随时创建和销毁，进程是可以占有系统的资源，cpu、内存空间、文件、输入输出设备等。它是系统进行资源分配和调度的一个独立单元。

线程，线程与进程类似，但是线程是比进程更小的执行单位，一个进程可以产生多个线程，与进程不同的是线程共享同一块内存空间和一组系统资源，所以线程之间的切换要比进程切换代价小的多，所以线程也被称之为轻量级进程。

## 你了解守护线程吗，它和用户线程的区别是什么？

java中存在两种线程，一种是守护线程，一种是用户线程。

任何线程都可以被设置为守护线程和用户线程，通过Thread.setDaemon\(boolean on\)，true表示设置为守护线程，false则为用户线程，设置用户线程一定要在start之前才能有效，两者唯一的区别是：

> 程序运行完毕时，JVM会等待用户线程完成后关闭，但是JVM不会等待守护线程，如果JVM只存在守护线程在运行，JVM随时会退出。

## 创建线程有哪些方式？

java创建线程有三种方式：  
1.继承Thread类创建线程类。  
2.实现Runnable接口创建线程类。  
3.通过Callable和Future创建线程。

## 线程存在哪些状态，每个状态都有什么特征？

线程有New、Runnable、Blocked、Wating、Time\_Wating、Terminated六个状态。

* New：初始化状态，线程被构建但是还没有被调用start方法。
* Runnable：运行状态，java线程将系统中的就绪和运行两种状态笼统的成为运行状态。
* Blocked：阻塞状态，表示线程阻塞于锁。
* Waiting：等待状态，进入该状态表示当前线程需要等待其他线程的通知或中断。
* Time\_Waitting：超时等待状态，该状态不同于Waiting，它是可以在指定时间自行返回的。
* Terminated：终止状态，表示当前线程已经执行完毕。

## start 和 run 方法有什么区别？

start和run，run是线程需要执行的方法，每个线程的需要执行的业务逻辑都需要写在run方法中，而start方法则是Thread的方法，是用来启动线程的，但是调用start后线程并不会立马执行，而是进入准备状态，等待cpu的调度。

## 如何使用 wait + notify 实现通知机制？

解决方案可以以某个对象为目标的阻塞，即利用 Object 类的 wait 和 notify方法实现线程阻塞与通知机制。

* 首先，wait、notify 方法是针对对象的，调用任意对象的 wait 方法都将导致线程阻塞，阻塞的同时也将释放该对象的锁，相应地，调用任意对象的 notify 方法则将随机解除该对象阻塞的线程，但它需要重新获取改对象的锁，直到获取成功才能往下执行。
* 其次，wait、notify 方法必须在 synchronized 块或方法中被调用，并且要保证同步块或方法的锁对象与调用 wait、notify 方法的对象是同一个，如此一来在调用 wait 之前当前线程就已经成功获取某对象的锁，执行 wait 阻塞后当前线程就将之前获取的对象锁释放。

## notify和notifyAll的区别？

notify是唤醒一个线程进行对象锁的争取，而notifyAll是唤醒所有等待中的线程进行锁争取，从源码细分的话，JVM每个java对象都有两个对应的队列，一个是entryList（锁池），一个是waitSet（等待池），每次调用notify的时候就会将waitSet队列的第一个线程放入entryList中然后进行锁的争取，而notifyAll则是将waitSet队列中所有的线程都放入entryList中进行锁争取。

## Thread类的 sleep 方法和对象的 wait 方法都可以让线程暂停执行，它们有什么区别？

sleep是Thread的静态方法，调用此方法会使当前线程暂停，将执行机会让给其他线程，但是对象的锁依旧保持，因此休眠时间结束后会自动恢复。 wait方法是object的方法，该方法是会导致当前线程释放对象的锁，线程暂停执行进入等待状态，此时线程会进入一个等待锁池（wait pool），只有其他线程调用对象的notify方法或者notifyAll方法时，该线程才会进入等待锁池（lock pool），如果线程重新获得锁就会继续执行。

## sleep、join、yield 方法有什么区别？

sleep是Thread静态方法，该方法直接可以让线程休眠指定时间，但是sleep休眠并不会使线程释放锁。

yield方法和sleep功能类似，但是yield与sleep不同的是该方法只是是线程回到可执行状态，并不是休眠线程，他只是礼让cpu的执行时间。

join方法，等待某个线程执行完毕，在join线程执行完成之前，调用线程是不能执行业务逻辑的。

## interrupt 、interrupted 和 isInterrupted 方法的区别？

interrupt是用来中断一个线程的方法，interrupted用来判断某个线程是否被中断，而且这个方法会清除线程的中断标志，连续调用两次第二次调用返回false，isInterrupted方法也是查询线程的中断标志的，但是它不会清楚中断标志。

## synchronized底层是如果实现的？

synchronized是java的同步关键字，底层是采用monitorenter和monitorexit两个指令去实现的，任何对象都有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态，monitorenter指令就是用来获取monitor所有权的操作，monitorexit则是对应释放monitor的所有权，monitor对象是同步的基本单元。 在早起java中，jdk1.6的monitor的实现完全是依靠系统内部的互斥锁，所以需要进行坏境切换，需要从用户态切换到内核态，所以早期的synchronized是一个重量级操作，在后期的jdk中，synchronized被不断的优化，当先的synchronized提供了三种不同的monitor实现，这三种分别是偏向锁、轻量级锁、重量级锁，大大改进了synchronized的性能。

所谓的锁升级、降级，就是jvm针对不同的并发坏境去优化synchronized的机制，当jvm检测到不同的竟态坏境的时候，就会自动的切换适合的锁去实现。

偏向锁：当没有竞争出现的时候，jvm会采用偏向锁的，jvm会使用cas去将线程的id设置到锁对象头中，以表示这个对象被当前线程获取到对象锁，所以偏向锁是并不涉及真正的互斥锁的，大部分的场景下一般都会被同一个线程获取到锁，使用偏向锁可以降低无竞争开销。

轻量级锁：如果有另外一个线程尝试获取一个已经被偏向过的锁对象，jvm就会撤销偏向锁，并切换到轻量级锁，表示当前线程出现了竟态，轻量级锁也是依赖cas的，cas尝试将对象锁的对象头中的线程id置为当前线程，如果cas成功，则直接使用轻量级锁，否则升级为重量级锁。

重量级锁：早期的synchronized的实现。

## java的对象头了解吗，主要由哪些部分组成。

在java中任何对象都有一个对象头，这个对象头主要用于存储对象的gc信息、锁信息、对象的hashcode、指向对象实例数据的指针以及数组长度，数组长度只针对于数组才会有的属性。综述对象头主要有三大块，Mark word、指向实例数据的指针、数组长度。

Mark word主要存储对象的hashcode、分代年龄和锁标记位，但是存储数据也不是一沉不变，运行期间会根据锁标志位变化而变化。 轻量级锁：指向栈锁记录的指针+锁标志位  
重量级锁：指向互斥量（重量级锁）的指针+锁标志位  
偏向锁：锁拥有线程id+epoch+对象分代年龄+锁标志位

指向对象实例的指针：32位虚拟机下是占用32bit，但是在64位虚拟机下占用是64bit，但是虚拟机开启了指针压缩的话会采用32bit。

数组长度：这个是数组对象独有的，占据内存大小为32bit。

## volatile关键字了解吗，它主要有哪些功能？底层具体是怎么实现的？

volatile是java的一个关键字，可以用来修饰变量，如果一个变量被修饰为volatile的话，那么JMM（java内存模型）保证所有线程看到的变量值都是一致的。

实现原理：被声明为volatile的变量，通过java字节码转化后，相比普通的变量被声明为volatile变量的汇编要多一个lock指令，那么这个lock指令在多核处理器下会引发两个事情，一个是将当前处理器缓存行的数据写会内存，一个是当一个缓存行所在的数据写回内存后就会导致其他的缓存的数据都会失效。

问题延伸：volatile这么做会引发什么问题，或者在性能方面会存在什么问题？答：我认为在性能方面，因为lock会引发数据强制写回缓存行导致其他缓存行数据失效，这种情况会使得性能有所降低，因为其他的缓存行数据在做计算的时候需要重新读取数据，但是即便是这种性能损失我认为相较于synchronized的也是高效许多。

## JMM了解吗，它是用来干啥的，为了解决什么问题？

JMM是java内存模型，它是虚拟机的一种规范，它屏蔽了各种硬件和操作系统的差异，规范了JVM如何解决java程序在各个平台下运行的变量可见性问题、原子性问题、有序性问题。

可见性问题：如何保证一个线程对共享变量的操作何时对其他线程可见。  
原子性问题：如何保证多个操作在cpu执行过程中不被中断。  
有序性问题：如何保证源代码编译不被重排序、指令级并行重排序、内存系统重排序。

线程之间的通信方式有两种，共享内存、消息传递，java采用的是共享内存模型。

**重排序**： 编译器会为了提供程序的运行效率，对程序源代码进行重排序，重排序 可以分为三种

* 编译器优化排序。编译器在不改变单线程程序的语义下，会对现有的代码重新安排语句执行顺序。    
* 指令级并行的重排序。现代处理器采用了指令并行技术来讲多条指令进行重叠执行，如果不存在数据依赖性，处理器可以改变对应的机器指令执行的顺序。
* 内存系统重排序。因为处理器使用了缓存和读/写缓冲区，这可能也使得原有的执行顺序被乱序执行。

**内存屏障**

* LoadLoad：如果指令为T1 LoadLoad T2，那么T1的数据的装载一定先于T2及后续的所有装载指令。
* StoreStore：如果指令为T1 StoreStore T2，那么T1的数据对其他处理器可见（刷新到内存中）一定先于T2及后续所有的存储指令。
* LoadStore：如果指令为T1 LoadStore T2，那么T1的数据装载一定先于T2及后续所有的存储指令刷新到内存中。
* StoreLoad：如果指令为T1 StoreLoad T2，那么T1的数据对其他处理器可见（刷新到内存中）一定先于T2的以及后续装载指令。StoreLoad指令还会将该屏障之前的所有内存访问（存储和装载）完成之后，才执行StoreLoad屏障之后的内存访问指令。

**happens-before规则**：

happens-before规则是对先行关系的描述，该规则表述的是如果一个操作A执行的结果需要对另外一个操作B可见，那么这两个操作必须存在happens-before关系，且A 一定是happens-before 于B，happens-before具体定义包括如下：  
1）程序顺序规则：一个线程中的每个操作happens-before与后续的任意操作。  
2）监视器锁规则：对一个锁的解锁happens-before于对这个锁的加锁。  
3）volatile规则：对一个volatile的写happens-before于任意后续对这个volatile域的读。  
4）传递性：如果A happens-before B B happens-before C 那么A也一定 happens-before C。  
5）start（）规则：线程start（）方法 happens-before 于此线程的每一个操作。  
6）join（）规则：线程中所有的操作都 happens-before 于线程的join方法（）。  
7）对象终结规则：一个对象的初始化完成（构造函数执行结束）happens-before 于该对象finalize方法。  
8）线程中断规则：对线程的中断方法（interrupt）的调用 happens-before 检测到中断事件的发生。

**volatile内存语义**：

volatile的内存语义分为读内存语义和写内存语义。 1. 读内存语义定义为：当线程读一个volatile会把该线程对应的本地内存置为失效，线程直接从主内存中取共享变量。 2. 写内存语义定义为：当线程写一个volatile变量的时候，JMM会把线程对应的本地内存的共享变量值刷新到主内存中。

volatile的内存语义具体实现。 1. 在每个volatile变量写的前面插入StoreStore屏障，保证volatile在写的时候前面所有的数据都已经先于当前变量写刷新至内存中。 2. 在每个volatile变量写的后面插入StoreLoad屏障，保证volatile写完之后会将变量刷新至主内存中，且先于后续的所有读操作。 3. 在每个volatile变量读的后面插入LoadLoad屏障，保证volatile读操作先于后续所有的普通读操作。 4. 在每个volatile变量读的后面插入LoadStore屏障，保证volatile读操作先于后续所有的写操作。

**锁的内存语义**：

当线程释放锁的时候，会把当前线程对应的本地内存中的共享变量刷回到主内存中。当线程获取锁的时候，JMM会把对应的本地内存置为失效，从而使得锁里面的代码必须从主内存中获取共享变量。在语义上和volatile语义相同。

锁释放-获取的内存语义实现至少有以下两种方式： 1. 利用volatile的写读所具有的内存语义。 2. 利用cas所附带的volatile读和写的内存语义。

**final内存语义**：

final修饰的字段变量需要遵循两个重排序规则。 1. 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。\(意思就是在对象引用被任意线程可见之前，对象的final域一定会被正确初始化\) 2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。（意思是读final域之前，一定会先去读这个final域的引用对象）

fina内存语义具体实现。

写： 1. JMM禁止编译器把final域的写重排序到构造函数之外。 2. 编译器会在写final域之后，构造函数return之前，插入一个StoreSto re屏障，这个屏障就是为了防止写final域操作被重排序到构造函数之外。

读： 1. 编译器会在读final域前面加一个LoadLoad屏障，防止了读对象引用和读final域不会被重排序。

## 什么情况下java程序会产生死锁，产生死锁的原因是因为什么？怎么解决死锁？

死锁是一种特定的程序状态，它是两个线程以上产生彼此依赖的循环等待中，比如线程1依赖线程2锁占用的锁，但是线程2又依赖线程1所占用的锁，彼此陷入漫长的等待中。在线上坏境中我们一般采用jstack来打印线程的线程栈信息，找出状态blocked的线程，以此来排查死锁具体原因。

那么在平时我们应该怎么避免死锁呢： 1. 平时我们在编程的时候应该提倡使用带超时时间的获取锁方法。 2. 如果必须要使用多个锁的情况下，在业务场景允许的情况尽量对多个锁使用顺序获取，比如某一个业务进行下来需要回去锁A、锁B，那么程序限定获取顺序为A到B，必须先获取A在获取B，以这种方式尽可能的减少死锁。 3. 业务允许的情况下尽量避免使用多锁，能够单锁解决的坚决不用两个锁去解决业务场景。

怎么解决死锁： 1. 重启应用，因为大多数死锁都是代码级问题。

## ThreadLocal你了解吗，它是用来干什么的？原理是什么？

答：ThreadLocal是jdk自带的类，那主要是提供线程访问本地变量，如果你创建了一个ThreadLocal，那么每个线程访问这个变量的时候都会在线程本地存一个该变量的副本，所有的操作都对副本进行操作。

原理：Thread类中有一个threadLocals和inheritableThreadLocals，他们都是ThreadLocalMap类型的变量，而ThreadLocalMap是一个定制化的map，默认情况下这两个变量都是为null，当线程第一次通过ThreadLocal访问set或者get方法的时候会对其进行初始化，每个线程的本地变量都是存放在这两个map中的，ThreadLocal其实就是一个工具壳，调用set的时候将值放入threadLocals或者inheritableThreadLocals中，调用get的时候从threadLocals或inheritableThreadLocals取出来，这两个都是map结构key为ThreadLocal对象，value为值，为什么使用map，因为一个线程需要绑定多个ThreadLocal。

**子类线程能访问父类线程的ThreadLocal变量吗？**

答：不能访问到，通过原理我们知道threadLocals和inheritableThreadLocals都是线程的变量，子类线程访问ThreadLocal变量的时候是从自己的threadLocals去取的，所以并不能取到父类线程的threadLocals中存储的值。

**怎么访问到父类线程中的ThreadLocal变量**

答：可以使用InheritableThreadLocal，因为Thread在创建的时候，构造器访问内部会执行一个init方法，该方法里面有一个逻辑就是默认情况下会将父类的inheritableThreadLocals拷贝至子类线程，所以使用InheritableThreadLocals的时候子类线程是可以访问到父类线程中的InheritableThreadLocals中的ThreadLocal变量，但是有一个小点需要注意一下因为是拷贝，所以父类改动本地InheritableThreadLocals中某个ThreadLocal变量的值的时候，子类线程是无法感知的。

## 什么是java共享变量内存可见性问题？

共享变量可见性问题，JMM规定所有的共享变量都存储在主内存中，每个线程需要使用变量的时候需要从主内存读取变量，并将变量复制到线程本地的工作空间进行计算，JMM是一个抽象概念，它是一组规范。实际中工作内存对应的是cpu的一级或者二级缓存，那么什么是共享变量可见性问题呢，假如现在有两个线程 线程A、线程B对变量C进行修改，A线程首次访问因为cpu的一级缓存和二级缓存都不存在变量C，那么A线程会加载变量C至一级缓存和二级缓存中，这个时候线程B进行变量C的读取，发现二级缓存存在则直接进行修改，并写会二级缓存，这个时候再切回线程A，这个时候A读取的变量则是旧数据，这就导致了可见性问题。

那么如何解决可见性问题呢，java中的volatile关键字就可以解决，或者采用锁机制也可以解决。

## 什么是伪共享？怎么解决？

当cpu访问某个变量的时候会对变量进行读取，先看缓存中是否存在，如果有则直接读取，没有的话则去主内存中读取该变量，并且把该变量所在的区域的一个Cache大小的内存数据读取到缓存中。假设这个时候有两个线程对这个Cache中的不同变量A、B进行修改，如果线程A修改了Cache中的变量A，那么就会导致线程B中的缓存被置为失效，线程B再次操作的时候则需要去二级缓存或者主内存中去重新获取数据，且线程A在对共享缓存行进行操作的时候，线程B只能等待，这会导致性能急剧下降，这就是伪共享问题。

**怎么解决伪共享问题？**  
其实伪共享最终原因是因为共享缓存行导致的，所以我们只需要将计算频率非常高的变量独享一个缓存行就行，在java8中有一个Contended注解，这个注解就可以解决这个问题，Contended注解是在变量前后加一个128字节宽度的空白填充，从而使得变量独占一个缓存行。

> 注意：一般cpu的缓存行大小为64字节宽。

## java并发包中的List有用过哪些吗？具体实现原理了解吗？

java并发包下的List只有一个CopyOnWriteArrayList，所以一般的情况会使用CopyOnWriteArrayList，也可以用Collections.synchronizedList\(\)来包装一个list从而达到线程安全，CopyOnWriteArrayList具体实现原理底层是基于独占锁ReentrantLock以及volatile修饰的数据实现的，每当有add、edit、remove操作的时候CopyOnWriteArrayList都会将原来的数组复制一份出来，对复制的数组进行修改，修改完成之后用新的数组替换原数组，进行add、edit、remove操作之前都会去获取独占锁，操作完成后释放独占锁，因为数组的引用是volatile的所以其他线程能够及时看到更新后的新数组。之所以采用这种方式是保证了读的并发数，保证写线程在操作的时候阻塞读线程。

注意点：CopyOnWriteArrayList是一个弱一致性的数组，如果有写线程在操作数组的时候，读线程读的还是老数据，所以存在数据有一定的延迟性。

## java并发包下的Set有用哪些？具体实现原理了解吗？

java并发包下的Set我了解到的有CopyOnWriteArraySet、ConcurrentSkipListSet。  
CopyOnWriteArraySet底层包装的是CopyOnWriteArrayList，它是基于CopyOnWriteArraySet实现的，没有做什么额外的逻辑。  
ConcurrentSkipListSet底层包装的是ConcurrentSkipListMap，ConcurrentSkipListMap底层则是基于跳表实现的。

## AQS了解吗，具体什么原理，java并发下都有哪些工具使用了AQS？

java.util.concurrent.locks.AbstractQueuedSynchronizer 抽象类，简称 AQS ，是一个用于构建锁和同步容器的同步器。事实上concurrent 包内许多类都是基于 AQS 构建。例如 ReentrantLock，Semaphore，CountDownLatch，ReentrantReadWriteLock，等。AQS 解决了在实现同步容器时设计的大量细节问题。

AQS使用了一个FIFO（先进先出）表示排队队列以及和一个volatile的state变量来实现并发相关的各种场景，队列中存的是线程节点，每个节点都维护了一个等待状态waitStatus。


### Java线程池创建有哪几种方式？

1. 采用Executors静态方法创建线程池。

**newFixedThreadPool(int nThreads)创建一个固定长度线程池，等待队列无限大**
**newCachedThreadPool(int nThreads)创建一个动态线程池，最大线程数无限大，核心线程为0，等待队列为SynchronousQueue**
**newScheduledThreadPool(int nThreads)创建一个固定长度线程池，且可以以延迟或定时方式来执行**

2. 直接new ThreadPoolExecutor创建一个线程池。

> 核心参数 corePoolSize（核心线程数）、maxnumPoolSize（最大线程数）、keepAliveTime（当线程数大于corePoolSize的空闲线程最大存活时间）、uni（时间单位）、workQueue（等待队列）、threadFactory（创建线程工厂）。

**有哪些拒绝策略**

1.AbortPolicy，直接抛出异常。<br>
2.CallerRunsPolicy，直接调用run方法执行。<br>
3.DiscardPolicy，抛弃任务。<br>
4.DiscardOldestPolicy，抛弃最老的任务。<br>

### 线程池关闭的几种方式？

1. shutdown（）方法，不会立即终止线程，但是会拒绝接受新的任务，等待缓存任务队列都执行完毕后才终止。
1. shutdownNow（）方法，立即终止线程，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

> 一般结合两个方法使用，可以先调用showdown方法，然后再调用awaitTermination等待线程终止，等待时间可以设置为60s，60s之后如果还没有终止则调用shutdownNow（）方法。

### execute和submit方法的区别？

execute提交的是实现Runnable的线程，且没有返回值，如果线程执行过程中有异常抛出则会直接抛出异常。<br>
submit提交的线程可以实现Callable也可以实现Runnable，存在返回值，如果线程执行过程中有异常抛出不会抛出到业务层，需要手动

### Fork/Join 框架是什么？

Fork/Join 框架是一个实现了 ExecutorService接口的多线程处理器。它可以把一个大的任务划分为若干个小的任务并发执行，充分利用可用的资源，进而提高应用的执行效率。

### 如何让一段程序并发的执行，并最终汇总结果？

1. 使用CountDownLatch：每个线程执行完毕都执行countDown方法，汇总结果的线程进行await等待。
2. 使用CyclicBarrier：让每个线程执行完毕后都执行await方法，并编写barrierCommand线程执行逻辑进行汇总。
3. 使用fork/join：

### Java并发队列都有哪些，具体原理是什么？

**1. ConcurrentLinkedQueue**

ConcurrentLinkedQueue是一个线程安全非阻塞队列，其底层采用单向链表实现，入队与出队操作使用CAS来实现线程安全。

