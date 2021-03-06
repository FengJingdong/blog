## Java并发特征


Java的并发采用的是共享内存模型，Java线程之间的通信总是隐式进行，整个通信过程对 程序员完全透明。如果编写多线程程序的Java程序员不理解隐式进行的线程之间通信的工作 机制，很可能会遇到各种奇怪的内存可见性问题。

* Java线程之间的通信由Java内存模型（本文简称为JMM）控制，JMM决定一个线程对共享 变量的写入何时对另一个线程可见。从抽象的角度来看，JMM定义了线程和主内存之间的抽 象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地 内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的 一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。

#### 指令序列的重排序

在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。重排序分3种类 型

* `编译器优化的重排序`。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
* `指令级并行的重排序`。现代处理器采用了指令级并行技术（Instruction-Level Parallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应 机器指令的执行顺序。 
* `内存系统的重排序`。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

过程：`源代码 -》 编译器优化的重排序 -》 指令级并行的重排序 -》 内存系统的重排序 -》 最终执行的指令序列`


* JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁
止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。

#### happens-before

从JDK 5开始，Java使用新的JSR-133内存模型。JSR-133使用happens-before的概念来阐述操作之间的内存可见性。在JMM中，如果一 个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。

happens-before规则如下。
* 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
* 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
* volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
* 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。

`注意` 两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个 操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一 个操作按顺序排在第二个操作之前

#### as-if-serial语义
as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程） 程序的执行结果不能被改变。编译器、runtime和处理器都必须遵守as-if-serial语义。

* 为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因 为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被 编译器和处理器重排序

#### volatile的内存语义
* 可见性。对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写 入。
* (伪)原子性：对任意单个volatile变量的读/写具有原子性，但类似于i++这种复合操作不 具有原子性。

#### volatile写和volatile读的内存语义

* 线程A写一个volatile变量，实质上是线程A向接下来将要读这个volatile变量的某个线程 发出了（其对共享变量所做修改的）消息。
* 线程B读一个volatile变量，实质上是线程B接收了之前某个线程发出的（在写这个volatile 变量之前对共享变量所做修改的）消息。
* 线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是线程A通过 主内存向线程B发送消息。

#### volatile内存语义的实现

* 为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来 禁止特定类型的处理器重排序。对于编译器来说，发现一个最优布置来最小化插入屏障的总 数几乎不可能。为此，JMM采取保守策略。下面是基于保守策略的JMM内存屏障插入策略。
* 在每个volatile写操作的前面插入一个StoreStore屏障。
* 在每个volatile写操作的后面插入一个StoreLoad屏障。 
(`StoreStore屏障可以保证在volatile写之前，其前面的所有普通写操作已经对任意处理器可见了。这是因为StoreStore屏障将保障上面所有的普通写在volatile写之前刷新到主
内存。volatile写后面的StoreLoad屏障,此屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排序。`)
* 在每个volatile读操作的后面插入一个LoadLoad屏障。
* 在每个volatile读操作的后面插入一个LoadStore屏障。(`LoadLoad屏障用来禁止处理器把上面的volatile读与下面的普通读重排序。 LoadStore屏障用来禁止处理器把上面的volatile读与下面的普通写重排序`)


