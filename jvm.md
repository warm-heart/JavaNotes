#  变量存储位置

## 局部变量

存在于方法中，所以在栈中，基本类型变量在栈中保存的直接是值，引用类型变量保存的是指向堆的地址。局部变量存储在栈中，当方法执行完后，局部变量随之消失。

## 成员变量

存在于类中，成员变量存储在堆中的对象里面

## 静态变量

 堆的Class对象中，类的实例对象有一个指针指向Class对象，故可以通过类的实例调用类的静态变量。

# java程序执行

![JavaLoad](C:/Users/wql/Desktop/JavaNotes-master/assets/JavaLoad.jpg)

如上图所示，首先Java源代码文件(.java后缀)会被Java编译器编译为字节码文件(.class后缀)，然后由JVM中的类加载器加载各个类的字节码文件，加载完毕之后，交由JVM执行引擎执行。在整个程序执行过程中，JVM会用一段空间来存储程序执行期间需要用到的数据和相关信息，这段空间一般被称作为Runtime Data Area（运行时数据区），也就是我们常说的JVM内存。因此，在Java中我们常常说到的内存管理就是针对这段空间进行管理（如何分配和回收内存空间）

# 方法区

方法区在JVM中也是一个非常重要的区域，它与堆一样，是被线程共享的区域。在方法区中，存储了每个类的信息（包括类的名称、方法信息、字段信息）、静态变量、常量以及编译器编译后的代码等。在Class文件中除了类的字段、方法、接口等描述信息外，还有一项信息是常量池，用来存储编译期间生成的字面量和符号引用。

JDK1.8中方法区的实现为元空间，在本地内存而不在JVM中，类加载过程生成的Class对象存储在堆中

# 堆

java中的堆是用来存储对象本身的以及数组（数组引用是存放在Java栈中的）。只不过和C语言中的不同，在Java中，程序员基本不用去关心空间释放的问题，Java的垃圾回收机制会自动进行处理。因此这部分空间也是Java垃圾收集器管理的主要区域。另外，堆是被所有线程共享的，在JVM中只有一个堆。

新生代  占总堆区的1/3 分为 Eden，from，to；比例为8：1：1

老年代 占总堆区的2/3

# Java栈

当java的方法执行时，会在栈中分配内存，Java栈中存放的是一个个的栈帧，每个栈帧对应一个被调用的方法，在栈帧中包括局部变量表(Local Variables)、操作数栈(Operand Stack)、指向当前方法所属的类的运行时常量池（运行时常量池的概念在方法区部分会谈到）的引用(Reference to runtime constant pool)、方法返回地址(Return Address)和一些额外的附加信息。当线程执行一个方法时，就会随之创建一个对应的栈帧，并将建立的栈帧压栈。当方法执行完毕之后，便会将栈帧出栈。下图表示了一个Java栈的模型：

局部变量表用来存储方法中的局部变量（包括在方法中声明的非静态变量以及函数形参）。对于基本数据类型的变量，则直接存储它的值，对于引用类型的变量，则存的是指向对象的引用。局部变量表的大小在编译器就可以确定其大小了，因此在程序执行期间局部变量表的大小是不会改变的

![JavaStatck](C:/Users/wql/Desktop/JavaNotes-master/assets/JavaStatck.jpg)

# 类加载

JVM把class文件加载到内存，并对数据进行校验、准备、解析、初始化，最终形成JVM可以直接使用的Java类型的过程。‘

## 加载过程

### 加载

   "加载"是"类加载"这个过程的一个阶段，是 “类加载”过程中最先开始进行的操作，加载阶段，虚拟机需要完成三件事：
    1、 根据类的全限定名获取定义此类的二进制字节流；
    2、 将这个字节流代表的静态存储结构转换为方法区的运行时数据结构；
    3、 在堆中为这个类生成一个java.lang.Class对象。

Java的虚拟机规范并没有规定从哪里获取、怎样获取二进制字节流，这个阶段也是用户参与度最高的阶段，用户可以根据二进制文件的不同形式在自定义类加载器控制字节流的获取方式，比如成熟的二进制获取方式和类加载器有：
1、 从Zip包中读取二进制文件，比如常见的jar、war、ear包；
2、 运行时动态生成，比如动态代理技术，在java.lang.reflect.Proxy中，使用 ProxyGenerator.generateProxyClass为各种就接口生成形如"*$Proxy"的代理类的二进制字节流；
3、 从网络中获取，这种场景比较常见的是Applet应用；
4、 其他文件生产，比如jsp文件生成的二进制class文件；

数组的加载跟普通类型加载有所不同，因为数组本身不是通过类加载器加载产生的，数组类是虚拟机自动生成的，但是数组的类型是通过类加载器完成加载的，数组类的创建过程需要遵循以下规则：
1、 如果数组的类型是引用类型，则引用类型需要使用递归来进行加载，并且数组需要被加载该数组类型的类加载器的命名空间上进行标识；
2、 如果数组的类型不是引用类型，是基本数据类型，Java虚拟机将会把数组标记为与引导类加载器关联；
3、 数组的可见性与数组类型的可见性保持一致，如果数组类型是基本类型，则默认可见性为public。

### 验证

验证是类加载的第二个阶段，这个阶段也是持续时间最长(从阶段连续性来说)，这个阶段从加载开始进行，一直进行到解析阶段结束。验证是为了保证class文件中的内容是符合虚拟机规范的二进制字节流，防止通过执行一些不安全的二进制字节流而导致虚拟机奔溃。 

Java语言本身是安全的语言，它做了很多的安全校验，比如类型转换、非正常的分支语句跳转、不合法的名称定义等等。但是我们知道，Java虚拟机并不只是执行Java语言编译后的class文件，它可以执行所有的二进制字节流文件(只要符合文件规范)，所以我们不能保证其他的文件是合法的，所以需要进行一些安全校验，以保证虚拟机执行的代码是不会危害虚拟机本身安全的。从整体来看，类加载过程的验证阶段可以分为四个部分：文件格式验证、元数据验证、字节码验证和符号引用验证。

### 准备

准备阶段是正式为类变量分配内存并**设置类变量初始值**的阶段。为这些变量分配内存，这个阶段分配的内存仅包括类变量（static修饰），实例变量将会在对象初始化阶段随着对象一起分配在堆中。

**对于设置类变量初始值**  如 public static int a=123;

**此步骤中仅仅把a设为初始值0，在初始化阶段把a设为123**

**被 final修饰的static变量（常量）会直接赋值。**

### 解析

解析阶段是虚拟机将符号引用转化为直接引用的过程，符号引用在之前已经介绍过了，在class文件中以形如"CONSTANT_Class_info"、"CONSTANT_Fieldref_info"、"CONSTANT_Methodref_info"格式存在，
参考 https://www.zhihu.com/question/30300585/answer/51335493

### 初始化

#### 触发初始化条件

- new
- 反射
- main方法的主类
- 初始化类时发现父类没有初始化，则先触发父类的初始化
- 调用类的静态方法和静态字段。

注意以下几种情况不会执行类初始化：

1. 通过子类引用父类的静态字段，只会触发父类的初始化，而不会触发子类的初始化。
2. 定义对象数组，不会触发该类的初始化。
3. 通过类名获取Class对象，不会触发类的初始化。
4. 通过Class.forName加载指定类时，如果指定参数initialize为false时，也不会触发类初始化，其实这个参数是告诉虚拟机，是否要对类进行初始化。
5. 调用类的常量不会进行初始化，因为类的常量在准备阶段已经赋值
6. 调用ClassLoader.load()方法不会进行初始化，这也是ClassLoader与Class.forName()的区别

#### 流程

这个阶段主要是对**类变量初始化**。换句话说，只对static修饰的变量或语句进行初始化。（不会执行代码块。代码块只有new执行）

如果初始化一个类的时候，其父类尚未初始化，则优先初始化其父类。

如果同时包含多个静态变量和静态代码块，则按照自上而下的顺序依次执行。

### 卸载

- 该类的所有实例都已经被GC，也就是JVM钟放不存在该Class的任何实例
- 加载该类的ClassLoader已经被GC
- 该类的Java.lang.Class对象没有在任何地方被引用，如不能在任何地方通过反射访问该类的方法

### 类加载器

把类加载阶段的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作交给虚拟机之外的类加载器来完成。这样的好处在于，我们可以自行实现类加载器来加载其他格式的类，只要是二进制字节流就行，这就大大增强了加载器灵活性。系统自带的类加载器分为三种：

1. 启动类加载器。
2. 扩展类加载器。
3. 应用程序类加载器。

### 双亲委派机制

如果一个类加载器收到了类加载器的请求.它首先不会自己去尝试加载这个类.而是把这个请求委派给父加载器去完成.每个层次的类加载器都是如此.因此所有的加载请求最终都会传送到Bootstrap类加载器(启动类加载器)中.只有父类加载反馈自己无法加载这个请求(它的搜索范围中没有找到所需的类)时.子加载器才会尝试自己去加载。

双亲委派模型的优点：java类随着它的加载器一起具备了一种带有优先级的层次关系.

例如类java.lang.Object,它存放在rt.jar之中.无论哪一个类加载器都要加载这个类.最终都是双亲委派模型最顶端的Bootstrap类加载器去加载.因此Object类在程序的各种类加载器环境中都是同一个类.相反.如果没有使用双亲委派模型.由各个类加载器自行去加载的话.如果用户编写了一个称为“java.lang.Object”的类.并存放在程序的ClassPath中.那系统中将会出现多个不同的Object类.java类型体系中最基础的行为也就无法保证.应用程序也将会一片混乱.

#### JVM在搜索类的时候，又是如何判定两个class是相同的呢？

​    JVM在判定两个class是否相同时，不仅要判断两个类名是否相同，而且要判断是否由同一个类加载器实例加载的。只有两者同时满足的情况下，JVM才认为这两个class是相同的。就算两个class是同一份class字节码，如果被两个不同的ClassLoader实例所加载，JVM也会认为它们是两个不同class。比如网络上的一个Java类org.classloader.simple.NetClassLoaderSimple，javac编译之后生成字节码文件NetClassLoaderSimple.class，ClassLoaderA和ClassLoaderB这两个类加载器并读取了NetClassLoaderSimple.class文件，并分别定义出了java.lang.Class实例来表示这个类，对于JVM来说，它们是两个不同的实例对象，但它们确实是同一份字节码文件，如果试图将这个Class实例生成具体的对象进行转换时，就会抛运行时异常java.lang.ClassCaseException，提示这是两个不同的类型。现在通过实例来验证上述所描述的是否正确：

# JVM调优

## JVM参数

### 标准参数

各个JVM版本基本不变

-help:java-version 查看Java版本

-server -client

-version -showversion

-cp -classpath

### X参数

非标准参数，各个JVM版本有可能不同，但变化的比较小，可以java-version 查看属于哪种模式，下图中为混合模式

![1587281243157](C:/Users/wql/Desktop/JavaNotes-master/assets/1587281243157.png)

-Xint：解释执行

-Xcomp：第一次使用就编译本地代码

-Xmixed：混合模式：JVM自己来决定是否编译称本地代码

### XX参数

使用最多，各个JVM版本有可能不同。

#### Boolean类型

+标识启用，-标识禁用

格式：-XX:[+-]<name>标识启用或禁用name属性

比如   -XX:+UseConcMarkSweepGc 启用CMS垃圾收集器

​         -XX:+UseG1GC 启用G1垃圾收集器

#### 非Boolean类型

格式： -XX:<name>=<value>表示name的属性值是value

比如： -XX:MaxGCPauseMillis=500  GC的最大停顿时间是500ms

​            XX:GCTimeRatio=19 ； -XX:MetaSpaceSize=32M  -XX:MaxMetaSpaceSize=32M

#### -Xmx -Xms

  属于XX参数

-Xms等价于 -XX:InitialHeapSize  初始化堆大小

-Xmx等价于 -XX:MaxHeapSize  最大的堆大小

## JVM运行时命令

- -xx:+PrintFlagsInitial  查看运行时JVM初始值
- -xx:+PrintFlagsFinal   查看运行时JVM最终值

### JPS

查看运行的java进程

```
C:\Users\wangqianlong>jps -l
72 com.example.ajax.AjaxApplication
```

### jinfo

 jinfo也是jvm中的一个命令，可以查看运行中JVM的全部参数，还可以设置部分参数。

#### 查看堆参数

```
C:\Users\wangqianlong>jinfo -flag MaxHeapSize 72
-XX:MaxHeapSize=2124414976
```

#### 设置参数

设置JVM的Boolean类型参数

```
查看PrintGC
C:\Users\wangqianlong>jinfo -flag PrintGC 11604
-XX:+PrintGC
//修改参数，关闭PrintGC
C:\Users\wangqianlong>jinfo -flag -PrintGC 11604
//查看修改后的参数
C:\Users\wangqianlong>jinfo -flag PrintGC 11604
-XX:-PrintGC
```

设置JVM的key，value类型的参数 命令jinfo -flag  name=value  pid

### jstat

查看JVM统计信息，垃圾回收信息，类加载信息，  

格式： options:-class,-complier,-gc,-printcompilation

#### 类加载

-class

```
C:\Users\wangqianlong>jstat -class 72（PID） 1000 10 //每隔1000ms输出一次，总共输出十次
Loaded  Bytes  Unloaded  Bytes     Time
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
  7503 13519.8        0     0.0       7.37
```

Loaded： 加载的类的个数

Bytes：加载了多个k

Unloaded：卸载的类的个数

Time：加载和卸载的时间

#### 垃圾回收

-gc

```
C:\Users\wangqianlong>jstat -gc 72 1000 5  //每隔1000ms输出一次，总共输出5次
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
14336.0 14336.0  0.0    0.0   238592.0 220468.6  65536.0    18290.7   35456.0 33736.6 4992.0 4557.2      8    0.141   2      0.108    0.249
14336.0 14336.0  0.0    0.0   238592.0 220468.6  65536.0    18290.7   35456.0 33736.6 4992.0 4557.2      8    0.141   2      0.108    0.249
14336.0 14336.0  0.0    0.0   238592.0 220468.6  65536.0    18290.7   35456.0 33736.6 4992.0 4557.2      8    0.141   2      0.108    0.249
14336.0 14336.0  0.0    0.0   238592.0 221661.6  65536.0    18290.7   35456.0 33736.6 4992.0 4557.2      8    0.141   2      0.108    0.249
14336.0 14336.0  0.0    0.0   238592.0 221661.6  65536.0    18290.7   35456.0 33736.6 4992.0 4557.2      8    0.141   2      0.108    0.249
```

结果最后一个字母如果是C代表Current，为总容量；如果是U代表Used，为使用的大小；单位KB

结果第一个字母代表堆的分区名称

S0C，S1C，S0U，S1U：S0与S1的总量与使用量

EC：Eden区总容量；EU：Eden区使用的容量

OC：old区总容量；OU：old区使用的容量

MC：元空间（MetaSpace）总容量 ；MU；MetaSpace使用的容量

CCSC、CCSU：压缩类空间总量与使用量

YGC、YGCT：YoungGc的次数与时间

FGC、YGCT：YoungGc的次数与时间

GCT：总的GC时间

#### JIT编译

-complier、-printcompilation

```
C:\Users\wangqianlong>jstat -compiler 72
Compiled Failed Invalid   Time   FailedType FailedMethod
    3871      0       0     1.41          0
```

Compiled：编译成功个数；Failed：编译失败个数；Invalid：无效的；Time：总花费时间

### jmap

#### 导出OOM文件

分析OOM原因时导出oom文件

jmap -dump：format=b,file=oom.hprof 72

在C:\Users\wangqianlong目录下生成一个oom.hprof文件

```
C:\Users\wangqianlong>jmap -dump:format=b,file=oom.hprog 72
Dumping heap to C:\Users\wangqianlong\oom.hprof ...
Heap dump file created
```

#### 查看堆的内存分配

jmap -heap 72

```
C:\Users\wangqianlong>jmap -heap 1736
Attaching to process ID 1736, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.161-b12

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 33554432 (32.0MB)
   NewSize                  = 11010048 (10.5MB)
   MaxNewSize               = 11010048 (10.5MB)
   OldSize                  = 22544384 (21.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 8388608 (8.0MB)
   used     = 5115576 (4.878593444824219MB)
   free     = 3273032 (3.1214065551757812MB)
   60.982418060302734% used
From Space:
   capacity = 1048576 (1.0MB)
   used     = 131072 (0.125MB)
   free     = 917504 (0.875MB)
   12.5% used
To Space:
   capacity = 1048576 (1.0MB)
   used     = 0 (0.0MB)
   free     = 1048576 (1.0MB)
   0.0% used
PS Old Generation
   capacity = 22544384 (21.5MB)
   used     = 16228000 (15.476226806640625MB)
   free     = 6316384 (6.023773193359375MB)
   71.98245026344476% used

16217 interned Strings occupying 2178280 bytes.
```

### jstack

打印JVM内部所有的线程， 如果有cpu利用率飙升，有可能发生死循环或者死锁。可以利用jstack命令找到线程并分析。

jstack 72（pid）

# 垃圾回收

## 可达性分析算法

从根节点开始往下搜索，没有搜索到的对象为可回收对象，也就是该对象到根节点不可达。

根节点：类加载器、Thread、虚拟机栈的本地变量表、static成员变量、常量引用、本地方法栈的变量。

## 垃圾回收算法

### 标记-清除

标记阶段标记要回收的垃圾，清除阶段直接清除垃圾，会产生内存碎片。

**缺点：**如果碎片太多，下一次为对象分配内存空间如果没有足够的连续内存就会出发GC。

### 复制

年轻代回收，把存活的对象移动到另一部分内存中，然后把已使用的内存空间  全部清空

**缺点：**实现简单，运行高效，空间利用率低，多出一块不用的内存。

### 标记-整理 

标记阶段标记要回收的垃圾，整理阶段把存活的对象向内存的一端移动，然后清理掉垃圾，不会产生内存碎片。

**缺点：**整理阶段比较耗时。

### 分代收集算法

分代收集算法并没有多么高深的原理，只是将前面的算法进行了合理组合与利用。根据对象存活周期的不同将内存划分为几块，一般将Java堆分为新生代和老年代。在新生代中，每次垃圾收集时会有大量的对象死去，只有少量存活，使用复制收集算法；老年代中对象存活率较高，没有额外空间对其进行分配担保，使用标记-清除算法或标记-整理来进行回收。


## JVM垃圾回收

- Young区采用复制算法。Young区分为survivor区（S0+S1）和Eden区

- Old区采用标记-清除 或者标记-整理。


### 对象分配

####  Eden区对象

对象优先在Eden区分配，即创建对象时现在Eden区分配。

#### 老年代对象

##### -XX:PretenureSizeThreshold

大对象直接进入老年代：JVM参数决定 -XX:PretenureSizeThreshold，如果超过这个参数的大小直接进入old区。

##### -XX:MaxTenuringThreshold

长期存活的对象进入老年代：每经过一次YoungGC依旧存活的对象年龄+1，等年龄达到-XX:MaxTenuringThreshold时进入old区。这个参数的大小默认为15，即15次YoungGC后依然存活进入old区。

##### -XX:TargetSurvivorRatio

当YoungGC后Survico区存活对象的比例达到-XX:TargetSurvivorRatio时，默认50%，YoungGC后当存活的对象超过50%后，计算剩余对象各年龄比例，要和MaxTenuringThreshold的值进行比较，以此保证MaxTenuringThreshold设置太大，导致对象无法晋升。

场景分析：

1. MaxTenuringThreshold为15
2. 年龄1的对象占用了33%
3. 年龄2的对象占用33%
4. 年龄3的对象占用34%。
5. 通过-XX:TargetSurvivorRatio比率来计算一个期望值，desired_survivor_size 。
6. 然后用一个total计数器，累加每个年龄段对象大小的总和。
7. 当total大于desired_survivor_size 停止。
8. 然后用当前age和MaxTenuringThreshold 对比找出最小值作为结果 

总体表征就是，年龄从小到大进行累加，当加入某个年龄段后，累加和超过**survivor**区域-XX:TargetSurvivorRatio的时候，就从这个年龄段往上的年龄的对象进行晋升。

年龄1的占用了33%，年龄2的占用了33%，累加和超过默认的-XX:TargetSurvivorRatio（50%），年龄2和年龄3的对象都会晋升至old区。

##### -XX:+PrintTrnuringDistribution  

发生YoungGC时YoungGC时打印对象的年龄分布情况。 方便查看存活的对象都是多少岁的。

# 垃圾收集器

## 停顿时间、吞吐量

**停顿时间：**垃圾收集器在做垃圾回收中断应用执行的时间。-XX:MaxGCPauseMillis

**吞吐量：**花在垃圾收集的时间和花在应用时间的占比。-XX:GCTimeRatio=<n>,垃圾收集时间占：1/1+n

## 串行收集器 Serial:Serial、Serial Old

概念：垃圾收集线程线程串行执行 

开启串行收集器：-XX:+UseSerialGc(作用与新生代)   -XX:+UseSerialOldGc（作用于老年带）

## 并行收集器 Parallel:Parallel Scavenge、Parallel Old 

吞吐量优先，Server模式下默认的收集器

概念： 指多条垃圾收集线程并行工作，但此用户线程仍处于等待状态，适用于科学计算，后台处理等若交互场景。

开启串行收集器：-XX:+UseParallelGc(作用与新生代)   -XX:+UseParallelOldGc（作用于老年带）

## 并发收集器Concurrent:CMS、G1

**响应时间优先。**

概念：用户线程和垃圾线程同时执行（但不一定时并行的，可能会交替执行）垃圾收集线程在执行的时候不会停顿用户程序的运行。适合对响应时间有要求的场景，比如Web。

## CMS垃圾收集器

- 作用于老年代

- 采用标记清除算法


与CMS收集器适配的新生代收集器，如果开启CMS，新生代默认为UseParNewlGC收集器

开启CMS收集器  -XX:+UseConcMarkSweepGC   -XX:+UseParNewlGC

### 收集过程

三色标记算法，开始时所有对象为白色，有至少一个引用未进行可达性分析置为灰色，如果引用全部分析并且自身不是垃圾置为黑色，垃圾回收时只有黑色和白色，白色为垃圾回收掉。

#### 初始标记

（STW）初始标记：这个阶段是标记从GcRoots直接可达的老年代对象、新生代引用的老年代对象，对象置为灰色。这个过程是单线程的。

#### 并发标记

由上一个阶段标记过的对象，开始tracing过程，标记所有可达的对象，这个阶段垃圾回收线程和应用线程同时运行。在并发标记过程中，应用线程还在跑，因此会导致有些对象会从新生代晋升到老年代、有些老年代的对象引用会被改变、有些对象会直接分配到老年代，这些受到影响的老年代对象所在的card会被标记为dirty，用于重新标记阶段扫描。

#### 并发预清理

预清理，也是用于标记老年代存活的对象，目的是为了让重新标记阶段的STW尽可能短。这个阶段的目标是在并发标记阶段被应用线程影响到的老年代对象，包括：（1）老年代中card为dirty的对象；（2）幸存区(from和to)中引用的老年代对象。因此，这个阶段也需要扫描新生代+老年代。【PS：会不会扫描Eden区的对象，我看源代码猜测是没有，还需要继续求证】

#### 重新标记

重新扫描堆中的对象，进行可达性分析,标记活着的对象。这个阶段扫描的目标是：新生代的对象 + Gc Roots + 前面被标记为dirty的card对应的老年代对象。如果预清理的工作没做好，这一步扫描新生代的时候就会花很多时间，导致这个阶段的停顿时间过长。这个过程是多线程的。

#### 并发清除

用户线程被重新激活，同时将那些未被标记为存活的对象标记为不可达；

#### 并发重置

CMS内部重置回收器状态，准备进入下一个并发回收周期。

### CMS出现的问题

#### 内存碎片

-XX:+UseCMSCompactAtFullCollection：FullGC之后对内存做一次压缩，减少内存碎片。

-XX:+CMSFullGCsBeforeCompaction：多少次FullGC之后压缩一次，因为压缩比较消耗时间

#### concurrent mode failure

由于CMS工作过程中用户线程也能工作，如果此时发生年轻代对象往老年代晋升或者直接在老年代分配对象时内存不足，则会出现concurrent mode failure，那么CMS会进行Serial-Old收集（兜底措施）。解决方案就是让old区剩余空间尽量大一些。-XX:CMSInitiatingOccupancyFraction：（Old区占有对象超过此参数触发FullGC），可以通过设置此参数，如果此参数过小，则会频繁发生CMS GC，如果过大，则会可能倒是old无法在并发收集过程中由于内存过小无法分配对象导致Serial-Old。

### 收集日志

#### ParNew日志

```
parNew收集过程
2020-12-03T10:46:10.991+0800: 13.044: [GC (Allocation Failure) 2020-12-03T10:46:10.991+0800: 13.044: [ParNew
Desired survivor size 17891328 bytes, new threshold 1 (max 6)
- age   1:   20141216 bytes,   20141216 total
- age   2:    5061032 bytes,   25202248 total
: 314559K->30258K(314560K), 0.0258068 secs] 380435K->106613K(1013632K), 0.0258880 secs] [Times: user=0.11 sys=0.00, real=0.03 secs]
解析:
[314559K->30258K(314560K), 0.0258068 secs] GC前新生代占用大小，GC后新生代占用大小，新生带总大小
[380435K->106613K(1013632K), 0.0258880 secs] GC前堆占用大小，GC后堆占用大小，堆总大小
```

#### CMS日志

```
//第一阶段 初始标记，CMS的第一个STW阶段，这个阶段会所有的GC Roots进行标记。
2020-12-03T10:46:10.715+0800: 12.768: [GC (CMS Initial Mark) [1 CMS-initial-mark: 65875K(699072K)] 361696K(1013632K), 0.0172504 secs] [Times: user=0.06 sys=0.00, real=0.02 secs]
解析：CMS Initial Mark 说明该阶段为初始标记阶段，65875K(699072K)当前老年代空间的用量和总量，361696K(1013632K)当前堆空间的用量和总量，0.0172504 secs初始化标记所耗费的时间。


//第二阶段并发标记
2020-12-03T10:46:10.732+0800: 12.785: [CMS-concurrent-mark-start]
2020-12-03T10:46:10.776+0800: 12.829: [CMS-concurrent-mark: 0.036/0.044 secs] [Times: user=0.20 sys=0.00, real=0.04 secs]
解析:CMS-concurrent-mark: 0.036/0.044 secs secs] 并发标记所所耗费的时间


//第三阶段 并发预清理阶段，并发执行的阶段。在本阶段，会查找前一阶段执行过程中,从新生代晋升或新分配或被更新的对象。通过并发地重新扫描这些对象，预清理阶段可以减少重新标记阶段的工作量。
2020-12-03T10:46:10.776+0800: 12.829: [CMS-concurrent-preclean-start]
2020-12-03T10:46:10.780+0800: 12.833: [CMS-concurrent-preclean: 0.004/0.004 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
解析： [CMS-concurrent-preclean: 0.004/0.004  secs] 预清阶段所使用功能的时间。


//第四阶段 并发可中止的预清理阶段。这个阶段工作和上一个阶段差不多。增加这一阶段是为了让我们可以控制这个阶段的结束时机，比如扫描多长时间（默认5秒）或者Eden区使用占比达到期望比例（默认50%）就结束本阶段。
2020-12-03T10:46:10.780+0800: 12.834: [CMS-concurrent-abortable-preclean-start]
2020-12-03T10:46:12.251+0800: 14.304: [CMS-concurrent-abortable-preclean: 1.200/1.470 secs] [Times: user=4.28 sys=0.22, real=1.47 secs]


//第五阶段 重新标记阶段，需要STW，从GC Root开始重新扫描整堆，标记存活的对象。需要注意的是，虽然CMS只回收老年代的垃圾对象，但是这个阶段依然需要扫描新生代，因为很多GC Root都在新生代。
2020-12-03T10:46:12.251+0800: 14.304: [GC (CMS Final Remark) [YG occupancy: 171873 K (314560 K)]2020-12-03T10:46:12.251+0800: 14.304: [Rescan (parallel) , 0.0161351 secs]2020-12-03T10:46:12.267+0800: 14.320: [weak refs processing, 0.0002245 secs]2020-12-03T10:46:12.267+0800: 14.321: [class unloading, 0.0090611 secs]2020-12-03T10:46:12.277+0800: 14.330: [scrub symbol table, 0.0159675 secs]2020-12-03T10:46:12.293+0800: 14.346: [scrub string table, 0.0008292 secs][1 CMS-remark: 76355K(699072K)] 248229K(1013632K), 0.0426435 secs] [Times: user=0.11 sys=0.00, real=0.04 secs]
解析：
[YG occupancy: 171873 K (314560 K)] =》 新生代空间占用大小，新生代总大小。
[Rescan (parallel) , 0.0161351 secs] =》 暂停用户线程的情况下完成对所有存活对象的标记，此阶段所花费的时间。
[weak refs processing, 0.0002245 secs] =》第一步 标记处理弱引用；
[class unloading, 0.0090611 secs] =》 第二步，标记那些已卸载未使用的类；
[scrub symbol table, 0.0159675 secs][scrub string table, 0.0008292 secs =》 最后标记未被引用的常量池对象。
[1 CMS-remark: 76355K(699072K)] 248229K(1013632K), 0.0426435 secs =》 重新标记完成后 老年代使用量与总量，堆空间使用量与总量。
[Times: user=0.11 sys=0.00, real=0.04 secs] =》 各个维度的时间消耗。


//第六阶段 并发清理阶段， 对前面标记的所有可回收对象进行回收
2020-12-03T10:46:12.294+0800: 14.347: [CMS-concurrent-sweep-start]
2020-12-03T10:46:12.313+0800: 14.367: [CMS-concurrent-sweep: 0.018/0.019 secs] [Times: user=0.13 sys=0.00, real=0.02 secs]
2020-12-03T10:46:12.314+0800: 14.367: [CMS-concurrent-reset-start]
2020-12-03T10:46:12.315+0800: 14.368: [CMS-concurrent-reset: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
解析：
[CMS-concurrent-sweep: 0.018/0.019 secs]  并发清理所耗费的时间。
[CMS-concurrent-reset: 0.001/0.001 secs]  重置数据和结构信息。
```

**调优：**

-XX:ConcGCThreads:并发的线程数，与用户线程一起执行的线程数。

-XX:+UseCMSCompactAtFullCollection：FullGC之后对内存做一次压缩，减少内存碎片。

-XX:+CMSFullGCsBeforeCompaction：多少次FullGC之后压缩一次，因为压缩比较消耗时间

-XX:CMSInitiatingOccupancyFraction：Old区占有对象超过此参数触发FullGC

-XX:+CMSScavengeBeforeRemark：FullGC之前先做YGC，建议打开此参数

## G1垃圾收集器

作用在新生代老年代，标记整理算法；开启G1收集器  -XX:+UseG1GC

G1收集器没有FullGC，而是MixedGC，回收所有的Young区和部分Old区

### G1日志

G1会有3种类型的GC，YGC（只会对新生代空间进行GC）、Miexd GC（混合GC，对新生代和部分老年代进行GC）、FGC（Full FC ，对整堆进行GC）。

#### **Young GC日志**

对新生代内存空间进行回收。

```text
 2020-10-20T20:06:24.845+0800: 3.927: [GC pause (G1 Evacuation Pause) (young), 0.0142232 secs]     //GC发生的系统时间，GC发生在程序启动后的多少秒后产生的GC。
 //该GC产生STW(由对象复制空间不足造成的STW）(发生在young区)，整个阶段的耗时。
  
  //下面是整个GC细节过程的耗时情况
  [Parallel Time: 12.9 ms, GC Workers: 8]        // 开启了8个GC线程            
       [GC Worker Start (ms): Min: 3926.9, Avg: 3934.2, Max: 3939.7, Diff: 12.8] // 工作线程启动时间
       [Ext Root Scanning (ms): Min: 0.0, Avg: 0.3, Max: 2.5, Diff: 2.5, Sum: 2.6] //扫描root的耗时
       [Update RS (ms): Min: 0.0, Avg: 0.3, Max: 2.1, Diff: 2.1, Sum: 2.1]    //更新RS的耗时
          [Processed Buffers: Min: 0, Avg: 1.9, Max: 15, Diff: 15, Sum: 15]   
       [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.0, Sum: 0.1]     //扫描RS的耗时
       [Code Root Scanning (ms): Min: 0.0, Avg: 0.1, Max: 0.6, Diff: 0.6, Sum: 0.6]  //
       [Object Copy (ms): Min: 0.0, Avg: 4.5, Max: 7.1, Diff: 7.1, Sum: 35.9] //对象拷贝耗时
       [Termination (ms): Min: 0.0, Avg: 0.4, Max: 0.5, Diff: 0.5, Sum: 2.9]
          [Termination Attempts: Min: 1, Avg: 74.2, Max: 133, Diff: 132, Sum: 594]
       [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1] 
       [GC Worker Total (ms): Min: 0.0, Avg: 5.5, Max: 12.8, Diff: 12.8, Sum: 44.3]
       [GC Worker End (ms): Min: 3939.7, Avg: 3939.7, Max: 3939.8, Diff: 0.0]
    [Code Root Fixup: 0.0 ms]
    [Code Root Purge: 0.0 ms]
    [Clear CT: 0.3 ms]
    [Other: 1.1 ms]          //其它任务的耗时
       [Choose CSet: 0.0 ms]  //CSet选择Region的时间
       [Ref Proc: 0.7 ms]    //处理对象引用的时间
       [Ref Enq: 0.0 ms]     //引用入ReferenceQueues队列的时间
       [Redirty Cards: 0.2 ms]
       [Humongous Register: 0.0 ms]
       [Humongous Reclaim: 0.0 ms]
       [Free CSet: 0.0 ms]               //释放CSet时间
    
   
   [Eden: 52.0M(52.0M)->0.0B(52.0M) Survivors: 8192.0K->8192.0K Heap: 74.2M(100.0M)->25.6M(100.0M)]
   Eden 回收前用量（总容量)->回收后用量(总容量),Survivors区回收前用量-回收后用量， 堆内存回收前用量（总容量）->回收后用量（总容量）
```

#### **Miexd GC日志**

Miexd GC回收整个新生代和部分老年代。

```text
 2020-10-20T20:06:25.068+0800: 4.151: [GC pause (Metadata GC Threshold) (young) (initial-mark), 0.0076494 secs]
 //GC产生STW(Metadata空间不足引起的GC）(initial-mark 说明整个GC属于混合GC，YGC和初始化标记阶段同时进行)。
 
 //下面和YGC一样，是整个GC细节过程的耗时情况,
    [Parallel Time: 6.1 ms, GC Workers: 8]
       [GC Worker Start (ms): Min: 4150.8, Avg: 4150.9, Max: 4151.0, Diff: 0.1]
       [Ext Root Scanning (ms): Min: 0.6, Avg: 2.3, Max: 5.8, Diff: 5.2, Sum: 18.3]
       [Update RS (ms): Min: 0.0, Avg: 0.5, Max: 0.8, Diff: 0.8, Sum: 4.2]
          [Processed Buffers: Min: 0, Avg: 2.2, Max: 7, Diff: 7, Sum: 18]
       [Scan RS (ms): Min: 0.0, Avg: 0.1, Max: 0.2, Diff: 0.2, Sum: 0.5]
       [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.3, Diff: 0.3, Sum: 0.3]
       [Object Copy (ms): Min: 0.0, Avg: 2.7, Max: 4.2, Diff: 4.2, Sum: 21.8]
       [Termination (ms): Min: 0.0, Avg: 0.2, Max: 0.2, Diff: 0.2, Sum: 1.3]
          [Termination Attempts: Min: 1, Avg: 4.6, Max: 11, Diff: 10, Sum: 37]
       [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
       [GC Worker Total (ms): Min: 5.8, Avg: 5.8, Max: 5.9, Diff: 0.1, Sum: 46.6]
       [GC Worker End (ms): Min: 4156.7, Avg: 4156.7, Max: 4156.7, Diff: 0.0]
    [Code Root Fixup: 0.0 ms]
    [Code Root Purge: 0.0 ms]
    [Clear CT: 0.2 ms]
    [Other: 1.3 ms]
       [Choose CSet: 0.0 ms]
       [Ref Proc: 1.0 ms]
       [Ref Enq: 0.0 ms]
       [Redirty Cards: 0.2 ms]
       [Humongous Register: 0.0 ms]
       [Humongous Reclaim: 0.0 ms]
       [Free CSet: 0.0 ms]
    [Eden: 50.0M(52.0M)->0.0B(53.0M) Survivors: 8192.0K->7168.0K Heap: 75.1M(100.0M)->28.6M(100.0M)]
 Heap after GC invocations=6 (full 0):
  garbage-first heap   total 102400K, used 29326K [0xdf000000, 0xdf100190, 0xe5400000)
   region size 1024K, 7 young (7168K), 7 survivors (7168K)
  Metaspace       used 16139K, capacity 16346K, committed 16384K, reserved 16688K
 }
  [Times: user=0.03 sys=0.00, real=0.01 secs] 
  
 
 //初始标记阶段STW
 2020-10-20T20:06:25.076+0800: 4.159: [GC concurrent-root-region-scan-start]
 2020-10-20T20:06:25.080+0800: 4.162: [GC concurrent-root-region-scan-end, 0.0039639 secs]
 
 //并发标记阶段
 2020-10-20T20:06:25.080+0800: 4.163: [GC concurrent-mark-start]
 2020-10-20T20:06:25.097+0800: 4.180: [GC concurrent-mark-end, 0.0170679 secs]
 
 //最终标记阶段STW
 2020-10-20T20:06:25.098+0800: 4.180: [GC remark 2020-10-20T20:06:25.098+0800: 4.180: [Finalize Marking, 0.0005518 secs] 2020-10-20T20:06:25.098+0800: 4.180: [GC ref-proc, 0.0003269 secs] 2020-10-20T20:06:25.098+0800: 4.181: [Unloading, 0.0040534 secs], 0.0052224 secs]
  [Times: user=0.02 sys=0.00, real=0.00 secs] 
  
  //筛选回收阶段STW
 2020-10-20T20:06:25.103+0800: 4.185: [GC cleanup 32M->30M(100M), 0.0008868 secs]
  [Times: user=0.00 sys=0.00, real=0.01 secs] 
 2020-10-20T20:06:25.104+0800: 4.186: [GC concurrent-cleanup-start]
 2020-10-20T20:06:25.104+0800: 4.186: [GC concurrent-cleanup-end, 0.0000233 secs]
  字段解析：
  [GC cleanup 32M->30M(100M), 0.0008868 secs] 回收前用量->回收之后用量（总容量）
```

#### **Full GC日志**

回收整个堆，包括新生代、老年代、元空间等 。

```text
 //回收前内存占情况
 Heap before GC invocations=297 (full 0):
  garbage-first heap   total 102400K, used 102278K [0xdf000000, 0xdf100190, 0xe5400000)
   region size 1024K, 0 young (0K), 0 survivors (0K)
  Metaspace       used 33404K, capacity 33769K, committed 33920K, reserved 34096K]
  字段解析：
   heap   total 102400K  used 102278K [0xdf000000, 0xdf100190, 0xe5400000)  => 堆空间总大小 已使用大小
 内存空间开始位置、已使用空间位置，结束位置。
 
   region size 1024K, 0 young (0K), 0 survivors (0K) => 单个region 大小，
   Metaspace       used 33404K, capacity 33769K, committed 33920K, reserved 34096K]
   
 //FGC 详情
 2020-10-20T20:06:34.447+0800: 13.529: [Full GC (Allocation Failure)  99M->74M(100M), 0.3039405 secs]
    [Eden: 0.0B(5120.0K)->0.0B(13.0M) Survivors: 0.0B->0.0B Heap: 99.9M(100.0M)->74.0M(100.0M)], [Metaspace: 33404K->33307K(34096K)]
  字段解析：
  [Full GC (Allocation Failure) =》 GC类型为Full GC, 原因是分配内存失败导致的Full GC。
   99M->74M(100M), 0.3039405 secs]  =》 回收前堆内空间占用量，回收后堆内存占用量（堆内存总量），耗费时间。
  [Eden: 0.0B(5120.0K)->0.0B(13.0M) =》 回收前eden区空间用量（总量），回收后Eden区空间用量（总量）。
   Survivors: 0.0B->0.0B  =》   回收前Survivorsn区空间用量），回收后Survivors区空间用量）。
   Heap: 99.9M(100.0M)->74.0M(100.0M)] =》 回收前堆空间用量（总量），回收后堆空间用量（总量）
 [Metaspace: 33404K->33307K(34096K)] =》 回收前元数据空间用量，回收后元数据空间用量（总量）
   
   
  //回收后内存占用情况
 Heap after GC invocations=298 (full 1):
  garbage-first heap   total 102400K, used 75785K [0xdf000000, 0xdf100190, 0xe5400000)
   region size 1024K, 0 young (0K), 0 survivors (0K)
  Metaspace       used 33307K, capacity 33649K, committed 33920K, reserved 34096K
 }
```



## 如何选择垃圾收集器

优先调整堆的大小让服务器自己来选择

如果内存小于100M，使用串行收集器

如果是单核，并且没有停顿时间要求，串行或者JVM自己来选。

如果允许停顿时间超过一秒，选择并行或者JVM自己来选。

如果响应时间最重要

 # GC

## 新生代GC（MinorGC/Young GC）

指发生在新生代的垃圾收集动作，因为 Java 对象大多都具备朝生夕灭的特性，所以 MinorGC 非常频繁，一般回收速度也比较快。

### MinorGC

从年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 **Minor GC**。这一定义既清晰又易于理解。但是，当发生Minor GC事件的时候，有一些有趣的地方需要注意到：

1. 当 JVM 无法为一个新的对象分配空间时会触发 Minor GC，比如当 Eden 区满了。所以分配率越高，越频繁执行 Minor GC。
2. 内存池被填满的时候，其中的内容全部会被复制，指针会从0开始跟踪空闲内存。Eden 和 Survivor 区进行了标记和复制操作，取代了经典的标记、扫描、压缩、清理操作。所以 Eden 和 Survivor 区不存在内存碎片。写指针总是停留在所使用内存池的顶部。
3. 执行 Minor GC 操作时，不会影响到永久代。从永久代到年轻代的引用被当成 GC roots，从年轻代到永久代的引用在标记阶段被直接忽略掉。
4. 质疑常规的认知，所有的 Minor GC 都会触发“全世界的暂停（stop-the-world）”，停止应用程序的线程。对于大部分应用程序，停顿导致的延迟都是可以忽略不计的。其中的真相就是，大部分 Eden 区中的对象都能被认为是垃圾，永远也不会被复制到 Survivor 区或者老年代空间。如果正好相反，Eden 区大部分新生对象不符合 GC 条件，Minor GC 执行时暂停的时间将会长很多。

#### MinorGC过程(Young GC)

​    虚拟机在进行MinorGC之前会判断老年代最大的可用连续空间是否大于新生代的所有对象总空间

​    1、如果大于的话，直接执行MinorGC

​    2、如果小于，判断是否开启HandlerPromotionFailure，没有开启直接FullGC

​    3、如果开启了HanlerPromotionFailure, JVM会判断老年代的最大连续内存空间是否大于历次晋升的大小，如果小于直接执行FullGC

​    4、如果大于的话，执行MinorGC



 ## 老年代GC（MajorGC）

指发生在老年代的 GC，出现了 MajorGC，经常会伴随至少一次的 MinorGC（但非绝对的，在 Parallel Scavenge 收集器的收集策略里就有直接进行 MajorGC 的策略选择过程）。MajorGC 的速度一般会比 MinorGC 慢 10 倍以上。

### FullGC

是清理整个堆空间—包括年轻代和永久代，指发生在老年代的 GC，出现了 **MajorGC**，经常会伴随至少一次的 **MinorGC**（但非绝对的，在 Parallel Scavenge 收集器的收集策略里就有直接进行 **MajorGC** 的策略选择过程）。**MajorGC** 的速度一般会比 **MinorGC** 慢 10 倍以上。

（1） 老年代空间不足

如果创建一个大对象，Eden区域当中放不下这个大对象，会直接保存在老年代当中，如果老年代空间也不足，就会触发**Full GC**。为了避免这种情况，最好就是不要创建太大的对象，大的数组。

（2）方法区空间不足

方法区空间满了。方法区中存放的为一些class的信息等，当系统中要加载的类、反射的类和调用的方法较多时，Permanet Generation可能会被占满，在未配置为采用CMS GC的情况下会执行Full GC。如果经过Full GC仍然回收不了，那么JVM会抛出错误信息：OOM 。为避免元空间占满造成Full GC现象，可采用的方法为增大Meta Space空间或转为使用CMS GC。

（3）Young GC出现promotion failure

promotion failure发生在Young GC, 如果Survivor区当中存活对象的年龄达到了设定值，会就将Survivor区当中的对象拷贝到老年代，如果老年代的空间不足，就会发生promotion failure， 接下去就会发生Full GC.

（4）统计**Young GC**发生时晋升到老年代的平均总大小大于老年代的空闲空间

在发生**Young GC**是会判断，是否安全，这里的安全指的是，当前老年代空间可以容纳**Young GC**晋升的对象的平均大小，如果不安全，就不会执行**Young GC**,转而执行**Full GC**。

（5）显式调用System.gc方法



# Object的**finalize**方法

如果对象在进行可达性分析后没有与GC Roots相连接的引用链，进行第一次标记；并且进行一次筛选：**此对象是否有必要执行finalize()方法**。任何一个对象的finalize()方法只会被系统自动调用一次，调用过finalize()方法而逃脱死亡的对象，第二次将不会再调用。

(1)没有必要执行的：对象已死，可以回收

对象没有覆盖 finalize() 方法
finalize() 方法已经被JVM调用过

(2) 有必要执行的情况

将有必要执行 finalize() 方法的对象放入F-Queue队列中
稍后由一个虚拟机自动建立的，低优先级的Finalizer线程去执行

- 如果对象在其finalize()方法中重新与引用链上任何一个对象建立关联，第二次标记时会将其移除**即将回收**的集合
- 否则，认为对象已死，可以回收

# OOM

```
static void digui() {
    digui();
}
public static void main(String[] args) {

    // OOM JAVA heap space
    List list = new ArrayList<>(1000000000);

    // OOM JAVA StackOverflowError
    digui();
}
```


