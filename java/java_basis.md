# java基本知识点

## 1.java有哪些基础数据类型

java有八种基本数据类型，每种基本类型如下：

* boolean 占据内存大小8bit 1字节，数值true  false。
* byte 占据内存大小8bit 1字节，数值范围在-2^7~2^7-1。
* int 占据内存大小32bit 4字节，数值范围在-2^31~2^31-1。
* short 占据内存大小16bit 2字节，数据范围在-2^15~2^15-1。
* char 占据内存大小16bit 2字节，数据范围在-2^15~2^15-1。
* float 占据内32bit 4字节，数据范围1.4 E-45~3.4028235 E38。
* double 占据内存64bit 8字节，数值范围 4.9 E-324~1.7976931348623157 E308。
* long 占据内存64bit 8字节，数值范围 -2^63~2^63-1。
* 
这里需要注意的是，java基础数据类型的包装类型有缓存机制，Boolean本身就是缓存因为只有两个值，Character是缓存的是0-127的值，Byte缓存的是-128-127，Short缓存的范围是-128-127，Integer的缓存值是可以修改的最低是-128，最高默认为127，但是最高可以配置，配置项为java.lang.Integer.IntegerCache.high。Long的缓存范围同Byte，Double和Float是不会缓存。

## 2.谈谈你对java平台的理解

java本身是一种面向对象的语言，他有两个显著的特点，1.跨平台性，一次编译，处处运行。2.垃圾回收机制，也是就常说gc，他通过gc回收已分配的内存，大部分情况下程序员无需操心内存的使用。 我们日常会接触到JRE或者JDK，JRE（java runtime Environment）也就是java运行坏境，JRE包含了JVM和java类库以及一些模块等，而JDK则可以看做是JRE的一个超集，JDK提供了更多的工具，比如编译器、诊断工具等。

诊断工具有：

* JPS （Java Virtual Machine Process Status Tool）类似于linux的ps命令，用来查看jvm进程状态信息。
* jstack 主要用来查看某个java进程内的线程堆栈信息，一般死锁会用到这个工具。
* jmap 用来查看堆内存使用情况，一般oom或者gc异常会用到这个工具。
* jstat 用来实时监控Java应用程序的资源和性能，也包含堆内存的gc信息。
* javap 用来反编译java类文件，可以查看java编译器生成的字节码。
* jprofiler 是一个商业授权的Java剖析工具，可对 CPU、堆、内存进行分析，功能强大。
* jcmd 用于向正在运行的JVM发送诊断命令请求。
* jconsole 图形化用户界面检测工具，用的比较少。
* jvisualvm java性能分析工具，也是图形化工具。
* Arthas 这是阿里开源的分析工具。

## 3.Exception和Error的区别

Exception和Error都继承了Throwable，在java中只有该类的实例可以被抛出或者捕获，它是异常处理机制的基本组成类型。不同点： Exception是程序运行中，可以去预料的意外情况，并且应该对其进行捕获并做相应的处理，Exception被分为可检查（checked）和不检查（unchecked）异常，可检查的在源代码里面必须对其显示捕获处理，这也是编译器检查的一部分，不检查异常通常是编码引起的逻辑错误，需要根据具体需要去做捕获，并不会在编译器进行强制要求。 Error是指在正常情况下，一般不可能出现的问题，属于错误，而且绝大多数Error会导致JVM处于不稳定、不可恢复状态。不需要进行捕获，常见比如OutOfMemoryError等。

## 4.谈谈final、finally、 finalize有什么不同

final是修饰符关键字，final可以用来修饰类、字段、方法，如果用来修饰类表示这个类不能被继承，如果用来修饰字段，表示这个字段的值不能被修改该字段的值为常量，如果用来修饰方法的话表示这个方法不能被重写。

finally是和try一起使用的，它是为了保证重点代码一定被执行的机制，配合try使用，可以用try-finally、try-catch-finally两种形式，一般用来释放重要的资源，例如jdbc连接，或者unlock锁等操作。

finalize是基础类Object的一个方法，它的设计是为了保证对象在垃圾回收之前完成特定的资源回收，finalize现在已经不被推荐使用，jdk9以后被置为deprecated。

## 5.强引用、软引用、弱引用、幻象引用有什么区别？

强引用：强引用就是我们平常最常见的普通对象引用，只要强引用还指向一个对象，就能表明对象还活着，JVM宁愿抛出OOM也不会去对该引用对象进行垃圾回收，只要强引用超过了作用域或者是显示的将相应的引用复制为null，就是可以被垃圾回收的，具体回收时间需要看垃圾回收策略。

软引用：通过SoftReference实现，SoftReference softReference = new SoftReference\(对象\)，一般软引用可以和ReferenceQueue配合起来使用，如果软引用对象被垃圾回收，JVM会把这个软引用加入到与之关联的引用队列中。JVM会保证出现OOM之前会回收软引用，软引用的具体回收时间是根据可用内存与SoftRefLRUPolicyMSPerMB值（该值默认为1000）的除值得到的。**应用场景：用来做内存敏感的缓存。**

弱引用：通过WeakReference实现，弱引用比软引用生命周期短，垃圾回收期一旦扫描到了弱引用，不管当前JVM内存是够足够，都会回收它的内存，由于垃圾回收是一个优先级较低的线程，所以弱引用不一定被立即回收，同样的弱引用也可以和ReferenceQueue配合使用，如果弱引用被垃圾回收器所回收就会把该引用放到ReferenceQueue中。**应用场景：同样可以用来内存敏感的缓存。**

虚引用：虚引用也称幻想引用，你不同通过虚引用去访问对象，虚引用仅仅是提供了一种确保对象被 finalize 以后，做某些事情的机制，如果一个引用仅仅持有虚引用那就和没有任何引用一样，随时可能被垃圾回收，虚引用可以配合ReferenceQueue使用，来达到引用对象内存被回收之前采取一些行动。**应用场景：跟踪对象被垃圾回收的活动**

## String、StringBuffer、StringBuilder有什么区别？

String是java非常基础且重要的类，用来管理和构造字符串，该类被声明为final类，不能被继承，而且所有属性也都是final的，所以一切针对String的拼接、裁剪等操作都会产生新的String对象。

StringBuffer是为了解决String类拼接、裁剪造成新对象的性能问题而出现的，StringBuffer本身是一个线程安全的类，内部使用了synchronized关键保证线程安全，但这种方式带来的也是性能的消耗。

StringBuilder是1.5版本后新增的，在功能上和StringBuffer没有本质区别，但是它去掉了线程安全的部分，在单线程下，性能优于StringBuffer，是绝大部分字符串拼接场景的首选。

## 动态代理是基于什么原理？

动态代理的实现方式有很多，首先可以用jdk本身提供的反射机制，该机制可以在程序运行时去直接操作类或者对象，通过该机制可以实现动态代理方法的调用等。其他的实现方式还有字节码操作，这个是直接对类文件的字节码进行修改从而达到动态代理的效果，现有的框架例如ASM、cglib（基于ASM）、javaassist等。

* 动态代理解决了什么问题，在你的业务中的应用场景是什么？

通过动态代理可以让调用者与实现者解耦，比如进行rpc调用，通过代理我们可以使rpc在调用者上使用更加简洁。

* 说说jdk的反射机制和cglib的区别，各自的具体实现方式是怎么样的？

首先，jdk的反射机制是通过实现对应的InvocationHandler，然后以代理目标的接口为纽带，为被代理的目标接口构建一个对象，进而应用程序就可以使用代理对象进行调用目标的逻辑。 cglib则不同，它是采用动态创建目标子类的方式，使用子类去达到近似使用被调用者本身的效果，在spring中默认采用的是jdk的动态代理，如需使用cglib需要进行显示指定。

* jdk的动态代理和cglib各自的优势是什么？

jdk的话，1.jdk本身支持，不需要引入额外的包，可能会比cglib更加可靠。2.可以进行jdk平滑的升级，但是cglib就不行，cglib需要保证在新的jdk版本上也能够继续使用。3. 代码实现简单。 cglib，1.调用目标不需要去额外的实现接口，从某种角度来交，强制需要调用目标实现接口是具有侵入性的，类似cglib就没有这个限制。2.高性能。

## @Autowired和@Resource的区别是什么？

两个注解都是用来装配依赖对象的，但是Autowired和Resource的区别在于： Autowired是spring内部注解，在默认情况下，必须要要对象存在，不能为null如果允许为null必须对其进行显示指定，required置为false，而且该注解是按照类型进行装配的，如果统一坏境下存在多个相同的类实例，需要进行显示指定（用Qualifier）装配其中一个实例，否则会报错。 Resource是jdk自带注解，该注解是按照name属性进行装配的，没有指定name的时候默认按照名称找，找不到依赖对象时，Resource注解会回退到按照类型装配，一旦指定了name属性就只能按照名称装配了。

## 对比Vector、ArrayList、LinkedList有何区别？

这三个类都实现了集合框架中的List集合，所以在功能上是相近的，但是因为底层设计不同，他们在安全、性能、行为方面有所不同。

**安全方面**：只有Vector是线程安全的，ArrayList和LinkedList都是线程不安全的，所以如果需要保证线程安全要么使用Vector，或者ArrayList和LinkedList采用Collections.synchronizedList去修饰或者使用CopyOnWriteArrayList。

**行为方面**：Vector和ArrayList底层都是采用的对象数组来保存对象，而LinkedList采用的是列表，所以Vector和ArrayList存在扩容机制，而且Vector扩容每次是变为原数组大小的两倍，ArrayList则是新增原来的百分之五十，LinkedList因为是列表所以不需要扩容。

**性能方面**：因为Vector底层采用加锁机制，所以它的性能是最低的，ArrayList和LinkedList在不同的操作方面性能各有优劣。

## 对比Hashtable、HashMap、TreeMap有什么不同？

这三个都是Map的实现类，都是key-value形式的键值对储存数据和操作数据，那么三个也是因为具体实现不一样从而在行为和性能上有所不同。

Hashtable是早期java的一个哈希表实现，本身是同步的，也就是线程安全的，key-value都不允许为null，因为加上了同步所以在性能方面要弱一些，因为底层是哈希表，所以存在扩容情况，每次扩容都会扩充为原来的两倍+1。

HashMap是应用更为广泛的哈希表实现，在功能上大致和Hashtable一致，不同的在于底层实现上，HashMap是非现场安全的，所以在性能上要高于Hashtable，但是在多线程下可能会出现异常情况，cpu暴增或者count不准确等，而且HashMap底层是采用哈希表+链表+红黑树来实现的，链表转红黑树有一个阈值，这个阈值是8，也就是同一个哈希桶上存在超过八个元素的时候会转为红黑树结构，HashMap允许null键和null值存储。

TreeMap是基于红黑树的一种提供顺序访问的Map，和HashMap不同，它的get、put、remove等操作都是O\(log\(n\)\)的时间复杂度，因为是红黑树，具体顺序要看指定的Comparator来决定，默认根据键的自然顺序来判断。

## 如何保证集合是线程安全的? ConcurrentHashMap如何实现高效地线程安全？

1.如何保证集合线程安全，要么使用jdk自带的线程安全的集合类，要么使用jdk提供的线程安全包装器（synchronized Wrapper），常见的线程安全集合有Vector、Hashtable、ConcurrentHashMap、CopyOnwriteArrayList、CopyOnwriteArraySet、ConcurrentSkipListSet、ConcurrentSkipListMap等，集合包装器有Collections.synchronizedSet或Collections.synchronizedList等等，但是包装器是使用粗粒度锁进行线程安全化，在性能上是不如jdk自带的线程安全集合的性能高。

2.ConcurrentHashMap如何高效的实现线程安全，早期：ConcurrentHashMap是采用分段锁去实现的，将内部的数据进行分段（Segment），里面则是HashEntry的数组，该数组和HashMap类似，Hash相同的数据也是采用链表去存放，内部分段数量（Segment）由concurrentcyLevel决定，默认是16，也可以创建的时候直接指定，但是java需要这个参数为2的幂次方，也就是传15也会变为16，底层采用2的幂次方主要是为了保证hash算法结果均匀分布。

**java8或以上的版本ConcurrentHashMap发生了改变，具体改变如下**

* 内部仍然存在Segment结构，但是Segment的只是为了保证序列化时兼容，并未有任何其他的功能。
* 因为不在使用Segment的，初始化采用了lazy-load懒加载模式，大大简化了操作步骤，避免初始化时候的不必要开销。
* 数据存储采用volatile实现可见性。
* 使用cas等操作实现特定场景下的无所并发操作，撞桶的情况下依旧采用了锁机制，但是摒弃了ReentrantLock锁而是直接采用了synchronized直接实现同步逻辑，因为synchronized的不断优化在性能上和ReentrantLock差异并不大，而且synchronized减少了内存消耗。
* 在size部分，使用类似LongAdder类思想进行极端情况优化。

## Java提供了哪些IO方式？ NIO如何实现多路复用？

java IO方式基于不同的io抽象模型和交互方式我们可以简单了分为三种：

1.基于java.io包下的传统io，它是基于流模型的，比如常见的file抽象、输入输出流等，交互方式是同步、阻塞的，也就是在读取和写的操作完成之前，线程会一直阻塞在哪里等待操作完成。

2.在java 1.4之后引入了NIO（java.nio包）框架，提供了channel、selector、Buffer等新的抽象，可以构建多路复用、同步非阻塞的io程序，同时提供了更接近操作系统底层的高性能数据操作方式。

3.在java 1.7之后，java对nio进行的优化，也就是NIO 2，引入了异步非阻塞方式，也被成为AIO。

## Java有几种文件拷贝方式？哪一种最高效？

Java 有多种比较典型的文件拷贝实现方式，比如：

利用 java.io 类库，直接为源文件构建一个 FileInputStream 读取，然后再为目标文件构建一个 FileOutputStream，完成写入工作。

```java
public static void copyFileByStream(File source, File dest) throws
        IOException {
    try (InputStream is = new FileInputStream(source);
         OutputStream os = new FileOutputStream(dest);){
        byte[] buffer = new byte[1024];
        int length;
        while ((length = is.read(buffer)) > 0) {
            os.write(buffer, 0, length);
        }
    }
 }
```

或者，利用 java.nio 类库提供的 transferTo 或 transferFrom 方法实现。

```java
public static void copyFileByChannel(File source, File dest) throws
        IOException {
    try (FileChannel sourceChannel = new FileInputStream(source)
            .getChannel();
         FileChannel targetChannel = new FileOutputStream(dest).getChannel
                 ();){
        for (long count = sourceChannel.size() ;count>0 ;) {
            long transferred = sourceChannel.transferTo(
                    sourceChannel.position(), count, targetChannel);            
                    sourceChannel.position(sourceChannel.position() + transferred);
            count -= transferred;
        }
    }
 }
```

当然，Java 标准类库本身已经提供了几种 Files.copy 的实现。对于 Copy的效率，这个其实与操作系统和配置等情况相关，总体上来说，NIO transferTo/From的方式可能更快，因为它更能利用现代操作系统底层机制，避免不必要拷贝和上下文切换。

## 谈谈接口和抽象类有什么区别？

接口和抽象类是 Java 面向对象设计的两个基础机制。

接口是对行为的抽象，它是抽象方法的集合，利用接口可以达到 API 定义和实现分离的目的。接口，不能实例化；不能包含任何非常量成员，任何 field 都是隐含着 public static final 的意义；同时，没有非静态方法实现，也就是说要么是抽象方法，要么是静态方法。Java 标准类库中，定义了非常多的接口，比如 java.util.List。

抽象类是不能实例化的类，用 abstract 关键字修饰 class，其目的主要是代码重用。除了不能实例化，形式上和一般的 Java 类并没有太大区别，可以有一个或者多个抽象方法，也可以没有抽象方法。抽象类大多用于抽取相关 Java 类的共用方法实现或者是共同成员变量，然后通过继承的方式达到代码复用的目的。Java 标准库中，比如 collection 框架，很多通用部分就被抽取成为抽象类，例如 java.util.AbstractList。

Java 类实现 interface 使用 implements 关键词，继承 abstract class 则是使用 extends 关键词，我们可以参考 Java 标准库中的 ArrayList。

