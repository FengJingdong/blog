## JVM运行时数据区

Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，已经创建和销毁时间，有的区域随着虚拟机进程的启动而创建，有些区域则依赖用户线程的启动和结束而创建和销毁。Java虚拟机所管理的内存将会包括以下几个运行时数据区域，如下图所示：

### 线程私有的数据区

#### 程序计数器
- 程序计数器（Program Counter Register）是一块较小的内存空间，它可以看做是当前线程所执行的字节码的行号指示器。字节码解释器工作时就是通过改变计数器的值来选取下一条需要执行的字节码指令、分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
- 由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的。在任何一个确定的时刻，一个处理器都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各个线程之间计数器互不影响，独立存储。
- 此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。
- 程序计数器是线程私有的，它的生命周期与线程相同（随线程而生，随线程而灭）。

#### Java虚拟机栈（JVM Stacks）

- 线程私有，描述Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。一个方法对应一个栈帧。JVM是基于栈的，所以每个方法从调用到执行结束，就对应着一个栈帧在虚拟机栈中入栈和出栈的整个过程。


 - 1. 局部变量表: 存放了编译期可知的各种基本类型、对象引用和returnAddress类型（指向了一条字节码指令地址）。其中64位长度long 和 double占两个局部变量空间，其他只占一个。值得注意的是：局部变量表所需的内存空间在编译期间完成分配。在方法运行的阶段是不会改变局部变量表的大小的。
- 2. 操作数栈: 虚拟机把操作数栈作为它的工作区，程序中的所有计算过程都是在借助于操作数栈来完成的，大多数指令都要从这里弹出数据，执行运算，然后把结果压回操作数栈。

- 3. 动态连接: 每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用（指向运行时常量池：在方法执行的过程中有可能需要用到类中的常量），持有这个引用是为了支持方法调用过程中的动态连接。
- 4. 方法出口: 当一个方法执行完毕之后，要返回之前调用它的地方，因此在栈帧中必须保存一个方法返回地址。
    - 规定的异常情况有两种：
        - 1.线程请求的栈的深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；
        - 2.如果虚拟机可以动态扩展，如果扩展时无法申请到足够的内存，就抛出OutOfMemoryError异常。
`设置JVM参数”-Xss228k”（栈大小为228k）。`

#### 本地方法栈（Native Method Stack）
    本地方法栈和虚拟机栈类似，不同的是虚拟机栈服务的是Java方法，而本地方法栈服务的是Native方法。在HotSpot虚拟机实现中是把本地方法栈和虚拟机栈合二为一的，同理它也会抛出StackOverflowError和OutOfMemoryError异常。

### 线程共有的数据区

#### 堆（Java Heap）
- Java虚拟机所管理的内存中最大的一块。由所有线程共享，在虚拟机启动时创建。
- 堆区唯一目的就是存放对象实例，几乎所有的对象实例都在这里进行分配。堆可以处于物理上不连续的内存空间，只要逻辑上是连续的就可以。 堆是垃圾收集器管理的主要区域。更好地回收内存。
- 堆中可细分为新生代和老年代，再细分可分为Eden空间、From Survivor空间、To Survivor空间。
- 在`JIT编译器`等技术的发展下，所有对象都在堆上进行分配已变得不那么绝对。有些对象实例也可以分配在栈中。
- 实现堆可以是固定大小的，也可以通过设置配置文件设置该为可扩展的。 如果堆上没有内存进行分配，并无法进行扩展时，将会抛出OutOfMemoryError异常。
> `设置JVM参数” -Xms20M -Xmx20M“`（前者表示初始堆大小20M，后者表示最大堆大小20M）

#### 方法区（Method Area）
- 在HotSpot上也被称为“永久代”（Permanent Generation），对应堆的新生代和老年代。方法区和堆的划分是JVM规范的定义，没有强制要求方法区必须实现垃圾回收，而不同虚拟机有不同实现，对于Hotspot虚拟机来说，将方法区纳入GC管理范围，JVM的垃圾收集器可以像管理堆区一样管理这部分区域，这样就不必单独管理方法区的内存，所以就有了”永久代“这么一说。
- 方法区和操作系统进程的正文段（Text Segment）的作用非常类似，它存储的是已被虚拟机加载的类信息、常量（从JDK7开始已经移至堆内存中）、静态变量、即时编译器编译后的代码等数据。对运行时常量池、常量、静态变量等数据做出了规定。
- 当方法区无法满足内存分配需求时，抛出OutOfMemoryError
>设置`JVM参数为”-XX:MaxPermSize=20M”`（方法区最大内存为20M）。

- 字符串常量池在JDK6的时候还是存放在方法区（永久代）,它会抛出OutOfMemoryError:Permanent Space；
- JDK7后则将字符串常量池移到了Java堆中，不会抛出OOM，若将堆内存改为20M则会抛出OutOfMemoryError:Java heap space；
- JDK8则是纯粹取消了方法区这个概念，取而代之的是”元空间（Metaspace）“，所以在JDK8中虚拟机参数”-XX:MaxPermSize”也就没有了任何意义，取代它的是`”-XX:MetaspaceSize“和”-XX:MaxMetaspaceSize”`等。

#### 运行时常量池（Runtime Constant Pool）

   它是方法区的一部分，Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项是常量池（Const Pool Table），用于存放编译期生成的各种字面量和符号引用。并非预置入Class文件中常量池的内容才进入方法运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的intern()方法。当方法区无法满足内存分配需求时，抛出OutOfMemoryError
