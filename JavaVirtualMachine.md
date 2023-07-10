[TOC]

# 1. JVM介绍

## 1.1 什么是JVM

> [Chapter 1. Introduction (oracle.com)](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-1.html)
>
> ## The Java Virtual Machine
>
> The Java Virtual Machine is the cornerstone of the Java platform. It is the component of the technology responsible for its hardware- and operating system-independence, the small size of its compiled code, and its ability to protect users from malicious programs.
>
> The Java Virtual Machine is an abstract computing machine. Like a real computing machine, it has an instruction set and manipulates various memory areas at run time. It is reasonably common to implement a programming language using a virtual machine; the best-known virtual machine may be the P-Code machine of UCSD Pascal.
>
> The first prototype implementation of the Java Virtual Machine, done at Sun Microsystems, Inc., emulated the Java Virtual Machine instruction set in software hosted by a handheld device that resembled a contemporary Personal Digital Assistant (PDA). Oracle's current implementations emulate the Java Virtual Machine on mobile, desktop and server devices, but the Java Virtual Machine does not assume any particular implementation technology, host hardware, or host operating system. It is not inherently interpreted, but can just as well be implemented by compiling its instruction set to that of a silicon CPU. It may also be implemented in microcode or directly in silicon.
>
> The Java Virtual Machine knows nothing of the Java programming language, only of a particular binary format, the file format. A file contains Java Virtual Machine instructions (or *bytecodes*) and a symbol table, as well as other ancillary information. `class``class`
>
> For the sake of security, the Java Virtual Machine imposes strong syntactic and structural constraints on the code in a file. However, any language with functionality that can be expressed in terms of a valid file can be hosted by the Java Virtual Machine. Attracted by a generally available, machine-independent platform, implementors of other languages can turn to the Java Virtual Machine as a delivery vehicle for their languages. `class``class`
>
> The Java Virtual Machine specified here is compatible with the Java SE 17 platform, and supports the Java programming language specified in *The Java Language Specification, Java SE 17 Edition*.

简单理解：

高级语言经过编译后，生成class二进制文件，同一份文件，可在不同平台上的JVM容器中运行。

![JVM](/JavaVirtualMachine.assets/JVM.jpg)



计算机组成结构：

- 输入设备

- 控制器

- 运算器

- 存储器

- 输出设备

  ![计算机组成结构](E:\Notes\JavaVirtualMachine.assets\计算机组成结构.jpg)

那么Java虚拟机对比物理机

- Class文件类比输入设备
- CPU指令集类比输出设备
- JVM类比存储器、控制器、运算器等

## 1.2 JVM产品

> 最常用的目前是HotSpot，可以通关`java -version`命令查看

**Oracle**：HotSpot、JRockit

**Ali**：TaobaoVM

**Zual**：Zing

## 1.3 JDK / JRE / JVM

> [Java Platform Standard Edition 8 Documentation (oracle.com)](https://docs.oracle.com/javase/8/docs/index.html)
>
> ![Java8_docs](E:\Notes\JavaVirtualMachine.assets\java8_docs.png)

## 1.4 结合JDK理解JVM

1. 能把Class文件翻译成不同平台的CPU指令集
2. 也是 Write Once Run Anywhere的保证

![Write Once Run Anywhere](E:\Notes\JavaVirtualMachine.assets\Write Once Run Anywhere.jpg)

## 1.5 HotSpot JVM Architecture

> [Getting Started with the G1 Garbage Collector (oracle.com)](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)

JVM 的主要组件包括类装入器（Class Loader）、运行时数据区域（Runtime Data Areas）和执行引擎（Execution Engine）。

![HotSpot JVM Architecture](E:\Notes\JavaVirtualMachine.assets\HotSpot JVM Architecture.PNG)

# 2. Class File

## 2.1 Java Source

> 可以运行于JVM的编程语言有Java、Kotlin、Groovy 等

```java
public class User {

    private Integer age;

    private String name = "zhangsan";

    private Double salary = 100.0;

    public void say() {
        System.out.println(name + "Say: ...");
    }

    public static Integer calc(Integer p1, Integer p2) {
        p1 = 3;
        Integer result = p1 + p2;
        return result;
    }

    public static void main(String[] args) {
        System.out.println(calc(1, 2));
    }
}
```

## 2.2 Early Compile

> [Compilation Overview (openjdk.org)](https://openjdk.org/groups/compiler/doc/compilation-overview/index.html)
>
> > The process of compiling a set of source files into a corresponding set of class files is not a simple one, but can be generally divided into three stages. Different parts of source files may proceed through the process at different rates, on an "as needed" basis.
> >
> > ![Compilation Overview](E:\Notes\JavaVirtualMachine.assets\Compilation Overview.png)
> >
> > This process is handled by the class.`JavaCompiler`
> >
> > 1. All the source files specified on the command line are read, parsed into syntax trees, and then all externally visible definitions are entered into the compiler's symbol tables.
> > 2. All appropriate annotation processors are called. If any annotation processors generate any new source or class files, the compilation is restarted, until no new files are created.
> > 3. Finally, the syntax trees created by the parser are analyzed and translated into class files. During the course of the analysis, references to additional classes may be found. The compiler will check the source and class path for these classes; if they are found on the source path, those files will be compiled as well, although they will not be subject to annotation processing.

Java文件编译过程包括两个阶段：

- 第一阶段是在编译阶段编译成Java字节码的过程，有些书籍中叫前端编译器，如Oracle的javac编译器；
- 第二阶段是在运行时，通过JVM的编译优化组件，对代码中的部分代码编译成本地代码，即JIT编译，如HotSpot中的C1、C2编译器（ Thus the threads used by client JIT compiler are called c1 compiler threads. Threads used by the server JIT compiler are called c2 compiler threads.）。

编译源码文件： User.java -> User.class

`javac .\User.java`

![Java Compile](E:\Notes\JavaVirtualMachine.assets\Java Compile.jpg)

## 2.3 Class Format

> [Chapter 4. The class File Format (oracle.com)](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html)

## 2.4 Analyse

> JVM加载解释映射，可参考 2.3 官网解释。

## 2.5 反汇编

> JDK自带命令 `javap -h`进行番编译，查看字节码信息和指令信息
>
> ```sh
> javap -v -c -p User.class > User.txt
> ```

JVM相对于class文件来说可以理解为操作系统；

Class文件相对与JVM来说可以理解为汇编语言或机器语言

# 3. 类加载机制

> [Chapter 5. Loading, Linking, and Initializing (oracle.com)](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html)
>
> > The Java Virtual Machine dynamically loads, links and initializes classes and interfaces. Loading is the process of finding the binary representation of a class or interface type with a particular name and *creating* a class or interface from that binary representation. Linking is the process of taking a class or interface and combining it into the run-time state of the Java Virtual Machine so that it can be executed. Initialization of a class or interface consists of executing the class or interface initialization method `<clinit>`

## 3.1 Loading



### 3.1.1 ClassLoader

- Bootstrap ClassLoader（启动类加载器）

  > 负责加载`$JAVA_HOME中jre/lib/rt.jar`里所有的class，加载`System.getProperty("sun.boot.class.path")`所指定的路径或jar。

- Extension ClassLoader（标准扩展类加载器） 

  > 负责加载java平台中扩展功能的一些jar包，
  >
  > 包括$JAVA_HOME中`jre/lib/*.jar`或`-Djava.ext.dirs`指定目录下的jar包，
  >
  > 加载`System.getProperty("java.ext.dirs")`所指定的路径或jar。

- Application ClassLoader（系统类加载器）

  > 负责加载ClassLoader中指定的jar包及目录中class 

- Custom ClassLoader（自定义加载器）

  > 属于应用程序根据自身需要自定义的ClassLoader，如Tomcat、JBoss都会根据j2ee规范自行实现。

![ClassLoader](E:\Notes\JavaVirtualMachine.assets\ClassLoader.jpg)

### 3.1.2 双亲委派机制

1. 检查某个类是否已经加载

   > ***自底向上***
   >
   > 从Custom ClassLoader到Bootstrap ClassLoader逐层检查，只要某个ClassLoader已加
   >
   > 载，就视为已加载此类，保证此类所有ClassLoader加载一次。

2. 加载

   > ***自顶向下***
   >
   > 也就是由上层来逐层尝试加载此类。

### 3.1.3 代码体验

```java
public static void main(String[] args) {
    // App ClassLoader
    System.out.println(User.class.getClassLoader());

    // Ext ClassLoader
    System.out.println(User.class.getClassLoader().getParent());

    // Bootstrap ClassLoader 原生C++语言实现，此处打印null
    System.out.println(User.class.getClassLoader().getParent().getParent());

    System.out.println(String.class.getClassLoader());
}
```

### 3.1.4 破坏双亲委派

#### 3.1.4.1 Tomcat

![Tomcat ClassLoader](E:\Notes\JavaVirtualMachine.assets\Tomcat ClassLoader.jpg)

#### 3.1.4.2 SPI机制

#### 3.1.4.3 OSGI

## 3.2 Linking

### 3.2.1 Verification

保证被加载类的正确性

### 3.2.2 Preparation

为类的静态变量分配内存，并将其初始化为默认值

### 3.2.3 Resolution

把类中的符号引用转换为直接引用

## 3.3 Initialization

对类的静态变量，静态代码块执行初始化操作

# 4. 运行时数据区

> [Chapter 2. The Structure of the Java Virtual Machine (oracle.com)](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-2.html#jvms-2.5)
>
> > The Java Virtual Machine defines various run-time data areas that are used during execution of a program. Some of these data areas are created on Java Virtual Machine start-up and are destroyed only when the Java Virtual Machine exits. Other data areas are per thread. Per-thread data areas are created when a thread is created and destroyed when the thread exits.

## 4.1 Method Area(方法区)

> JVM运行时数据区是一种规范，真正的实现实在JDK8中就是Metaspace，在JDK6或者7中就是Perm Space

方法区时各个线程共享的内存区域，在虚拟机启动时创建，虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的是与Java堆区分开来。

用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

### 4.1.1 常量池和运行时常量池

[The Constant Pool (oracle.com)](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html#jvms-4.4)

> Java Virtual Machine instructions do not rely on the run-time layout of classes, interfaces, class instances, or arrays. Instead, instructions refer to symbolic information in the table. `constant_pool`
>
> All table entries have the following general format: `constant_pool`

[Run-Time Constant Pool (oracle.com)](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-2.html#jvms-2.5.5)

> A *run-time constant pool* is a per-class or per-interface run-time representation of the table in a file ([§4.4](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html#jvms-4.4)). It contains several kinds of constants, ranging from numeric literals known at compile-time to method and field references that must be resolved at run-time. The run-time constant pool serves a function similar to that of a symbol table for a conventional programming language, although it contains a wider range of data than a typical symbol table. `constant_pool``class`
>
> Each run-time constant pool is allocated from the Java Virtual Machine's method area ([§2.5.4](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-2.html#jvms-2.5.4)). The run-time constant pool for a class or interface is constructed when the class or interface is created ([§5.3](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-5.html#jvms-5.3)) by the Java Virtual Machine.
>
> The following exceptional condition is associated with the construction of the run-time constant pool for a class or interface:
>
> - When creating a class or interface, if the construction of the run-time constant pool requires more memory than can be made available in the method area of the Java Virtual Machine, the Java Virtual Machine throws an . `OutOfMemoryError`
>
> See [§5 (*Loading, Linking, and Initializing*)](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-5.html) for information about the construction of the run-time constant pool.

### 4.1.2 String常量到底存在哪

```java
public class SCPDemo {
    public static void main(String[] args) {
        // 这个常量一定会放到字符串常量池
        String str1 = "SCP";
        String str2 = "SCP";
        String str3 = new String("SCP");
        // 找字符串常量池中是否有该常量，如果有就直接返回，没有就创建
        String str4 = str3.intern();
        
        // equals 只会比较值 == 会比较地址
        System.out.println(str1.equals(str2)); 	// true
        System.out.println(str1 == str2);       // true
        System.out.println(str1.equals(str3)); 	// true
        System.out.println(str1 == str3);       // false
        System.out.println(str1.equals(str4)); 	// true
        System.out.println(str1 == str4);       // true
    }
}
```

![String Constant Pool](E:\Notes\JavaVirtualMachine.assets\String Constant Pool.jpg)

## 4.2 Heap(堆)

> 1. Java堆是Java虚拟机所管理内存中最大的一块，在虚拟机启动时创建，被所有线程共享
> 2. Java对象实例以及数组都在堆上分配
> 3. 堆内存不足时，也会抛出OOM

### 4.2.1 Java对象内存布局

> 一个Java对象在内存中包括三个部分
>
> - 对象头
> - 实例数据
> - 对齐填充

![Heap Of Java Object](E:\Notes\JavaVirtualMachine.assets\Heap Of Java Object.jpg)

### 4.2.2 方法区引用指向堆

![方法区引用指向堆](E:\Notes\JavaVirtualMachine.assets\方法区引用指向堆.png)



### 4.2.3 堆指向方法区

![堆指向方法区](E:\Notes\JavaVirtualMachine.assets\堆指向方法区.png)

## 4.3 Java Virtual Machine Stacks(Java虚拟机栈)

> 1. 虚拟机栈是一个线程执行的区域，保存着一个线程中方法的调用状态。即一个Java线程的运行状态，由一个虚拟机栈来保存，所以虚拟机栈肯定是线程私有的，独有的，随着线程创建而创建。
> 2. 每一个被线程执行的方法，为该栈中的栈帧，即每个方法对应一个栈帧。调用一个方法，就会向栈中压入一个栈帧；一个方法调用完成，就会把该栈帧从栈中弹出。

### 4.3.1 代码示例

```java
void a() {
    b();
}
void b() {
    c();
}
void c() {
}
```

### 4.3.2 压栈出栈

![虚拟机栈](E:\Notes\JavaVirtualMachine.assets\虚拟机栈.png)

### 4.3.3 Frame(栈帧)

![Frame(栈帧)](E:\Notes\JavaVirtualMachine.assets\Frame(栈帧).png)



### 4.3.4 字节码

> 使用idea插件Show Bytecode查看class文件，查看calc方法
>
> 或者
>
> `javap -c .\User.class`
>
> `javap`命令查看 [2.1 Java Source](#2.1 Java Source) 编译后的字节码指令

```txt
  public static calc(Ljava/lang/Integer;Ljava/lang/Integer;)Ljava/lang/Integer;
   L0
    LINENUMBER 20 L0
    ICONST_3       // 将int类型常量3压入[操作数栈]
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
    ASTORE 0       // 将int类型存入[局部变量0]
   L1
    LINENUMBER 21 L1
    ALOAD 0		  // 从[局部变量0]中装载int类型值入栈
    INVOKEVIRTUAL java/lang/Integer.intValue ()I
    ALOAD 1		  // 从[局部变量1]中装载int类型值入栈
    INVOKEVIRTUAL java/lang/Integer.intValue ()I
    IADD		  // 将栈顶元素弹出栈，执行int类型的加法，结果入栈
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
    ASTORE 2       // 将栈顶int类型值保存到[局部变量2]中
   L2
    LINENUMBER 22 L2
    ALOAD 2		  // 从[局部变量2]中装载int类型值入栈
    ARETURN        // 从方法中返回int类型的数据
   L3
    LOCALVARIABLE p1 Ljava/lang/Integer; L0 L3 0
    LOCALVARIABLE p2 Ljava/lang/Integer; L0 L3 1
    LOCALVARIABLE result Ljava/lang/Integer; L2 L3 2
    MAXSTACK = 2
    MAXLOCALS = 3
```



### 4.3.5 Index为0还是1

对于Java虚拟机栈中的Local Variables，到底是从0开始还是1开始，要看当前方法是static还是实例方法。

> 1 The Java Virtual Machine uses local variables to pass parameters on method invocation. On class method invocation, any parameters are passed in consecutive local variables starting from local variable 0.
>
> 2 On instance method invocation, local variable 0 is always used to pass a reference to the object on which the instance method is being invoked (this in the Java programming language).
>
> 3 Any parameters are subsequently passed in consecutive local variables starting from local variable 1.

### 4.3.6 栈引用指向堆

![栈引用指向堆](E:\Notes\JavaVirtualMachine.assets\栈引用指向堆.png)

## 4.4 Native Method Stacks(本地方法栈)

![Native Method Stacks(本地方法栈)](E:\Notes\JavaVirtualMachine.assets\Native Method Stacks(本地方法栈).png)

## 4.5 The PC Register

如果线程正在执行Java方法，则计数器记录的是正在执行的虚拟机字节码指令的地址。

如果正在执行的是Native方法，则这个计数器为空。

# 5. 内存模型

## 5.1 整体描述

上面对运行时数据区描述了很多，其实重点存储数据的是堆和方法区（非堆），所以内存的设计也着重从这两方面展开（注意，这两块区域都是线程共享的）。对于虚拟机栈、程序计数器都是线程私有的。

- 一块是非堆区，一块是堆区
- 堆区分为两大块，一个是Old区，一个是Young区
- Young区分为两大块，一个是Survivor（S0+S1），一块是Eden区
- S0和S1一样大，也可以交From和To

![Java Memory Model Description](E:\Notes\JavaVirtualMachine.assets\Java Memory Model Description.png)

## 5.2 Java Memory Model

> [Chapter 17. Threads and Locks (oracle.com)](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)

## 5.3 对象在内存中的分配回收

![对象在内存中分配流程](E:\Notes\JavaVirtualMachine.assets\对象在内存中分配流程.png)

## 5.4 代码体验

### 5.4.1 堆的OOM

> `-Xmx50M -Xms50M`

```java
@RestController
public class HeapController {
    List<Person> list = new ArrayList<>();
    
    @GetMapping("/heap")
    public String heap() {
        while(true) {
            list.add(new Person());
        }
    }
}
```

### 5.4.2 方法区OOM

> `-XX:MetaspaceSize=50M -XX:MaxMetaspaceSize=50M`

1. asm依赖

   ```xml
   <dependency>
       <groupId>asm</groupId>
       <artifactId>asm</artifactId>
       <version>${asm.version}</version>
   </dependency>
   ```

2. 工具类

   ```java
   public class MyMetaspace extends ClassLoader {
       public static List<Class<?>> createClasses() {
           List<Class<?>> classes = new ArrayList<Class<?>>();
           for (int i = 0; i < 10000000; ++i) {
               ClassWriter cw = new ClassWriter(0);
               cw.visit(Opcodes.V1_1, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
               MethodVisitor mw = cw.visitMethod(Opcodes.ACC_PUBLIC, "<init>", "()V", null, null);
               mw.visitVarInsn(Opcodes.ALOAD, 0);
               mw.visitMethodInsn(Opcodes.INVOKESPECIAL, "java/lang/Object", "<init>", "()V");
               mw.visitInsn(Opcodes.RETURN);
               mw.visitMaxs(1, 1);
               mw.visitEnd();
               Metaspace test = new Metaspace();
               byte[] code = cw.toByteArray();
               Class<?> exampleClass = test.defineClass("Class" + i, code, 0, code.length);
               classes.add(exampleClass);
           }
           return classes;
       }
   }
   ```

3. 代码

   ```java
   @RestController
   public class NonHeapController {
       List<Class<?>> list=new ArrayList<Class<?>>();
       @GetMapping("/nonheap")
       public String nonheap() {
           while(true) {
               list.addAll(MyMetaspace.createClasses());
           }
       }
   }
   ```

### 5.4.3 栈溢出

```java
public class StackDemo {
    public static long count = 0;
    public static void method() {
        System.out.println(count++);
        method();
    }
    public static void main(String[] args) {
        method();
    }
}
```

# 6. 垃圾收集

## 6.1 确定垃圾对象

### 6.1.1 引用计数法

对于某个对象而言，只要应用程序中持有该对象的引用，就说明该对象不是垃圾，如果一个对象没有任何指针对其引用，它就是垃圾。

### 6.1.2 可达性分析

通过GC Root的对象，开始向下寻找，看某个对象是否可达。

> 能作为GC Root
>
> - 类加载器
> - Thread
> - 虚拟机栈的本地变量表
> - Static成员
> - 常量引用
> - 本地方法栈的变量
> - ...

![GC Root可达性分析](E:\Notes\JavaVirtualMachine.assets\GC Root可达性分析.png)

## 6.2 垃圾收集算法

### 6.2.1 标记 - 清除

- **标记：**找出内存中需要回收的对象，并且把它们标记出来
- **清除：**清除掉被标记需要回收的对象，释放出对应的内存空间

> **缺点：**
>
> 1. 标记和清除两个过程都比较耗时，效率不高
> 2. 会产生大量不连续的内存碎片，空间碎片太多，可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前出发一次垃圾收集操作

![GC-标记清除](E:\Notes\JavaVirtualMachine.assets\GC-标记清除.png)

### 6.2.2 标记 - 整理

- **标记：**找出内存中需要回收的对象，并且把它们标记出来

- **整理：**清除掉被标记需要回收的对象，释放出对应的内存空间，并整理存活对象

![GC-标记整理](E:\Notes\JavaVirtualMachine.assets\GC-标记整理.png)

### 6.2.3 标记 - 复制

- **标记：**找出内存中需要回收的对象，并且把它们标记出来

- **复制：**将存活对象复制至另一片空闲空间，将该空至空

> **缺点：**
>
> 1. 存在大量的复制操作，效率会降低
> 2. 空间利用率降低

![GC-标记复制](E:\Notes\JavaVirtualMachine.assets\GC-标记复制.png)

## 6.3 垃圾收集算法选择

**Young区：**复制算法（对象在被分配之后，可能生命周期比较短，Young区复制效率比较高）

**Old区：**标记清除或者标记整理（Old区对象存活时间比较长，复制来复制去没必要，不如做个标记再清理）

## 6.4 垃圾收集器

> 如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。

### 6.4.1 Serial

> 可以用与新老年代
>
> 新生代：复制算法
>
> 老年代：标记-整理算法

> The serial collector uses a single thread to perform all garbage collection work

![GC - Serial](E:\Notes\JavaVirtualMachine.assets\GC - Serial.png)

### 6.4.2 Parallel

> 可以用于新老年代
>
> **新生代：**复制算法
>
> **老年代：**标记整理算法

> The parallel collector is also known as throughput collector, it's a generational collector similar to the serial collector. The primary difference between the serial and parallel collectors is that the parallel collector has multiple threads that are used to speed up garbage collection

![GC - Parallel](E:\Notes\JavaVirtualMachine.assets\GC - Parallel.png)

### 6.4.3 CMS(ConcurrentMarkSweepGC)

>[Concurrent Mark Sweep (CMS) Collector (oracle.com)](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector)
>
>> The Concurrent Mark Sweep (CMS) collector is designed for applications that prefer shorter garbage collection pauses and that can afford to share processor resources with the garbage collector while the application is running. 
>
>可以用于新老年代
>
>采用标记-清除算法
>
>回收过程：[Getting Started with the G1 Garbage Collector (oracle.com)](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)
>
>> 1. The Concurrent Mark Sweep (CMS) collector (also referred to as the concurrent low pause collector) collects the tenured generation. It attempts to minimize the pauses due to garbage collection by doing most of the garbage collection work concurrently with the application threads. Normally the concurrent low pause collector does not copy or compact the live objects. A garbage collection is done without moving the live objects. If fragmentation becomes a problem, allocate a larger heap.
>>
>> 2. **Note:** CMS collector on young generation uses the same algorithm as that of the parallel collector.

1. **初始标记：**CMS initial mark 标记GC Roots直接关联对象，不用Tracing，速度很快
2. **并发标记：**CMS concurrent mark 进行GC Roots Tracing
3. **重新标记：**CMS remark 修改并发标记因用户程序变动的内容
4. **并发清除：**CMS concurrent sweep 清除不可达对象回收空间，同时有新垃圾产生，留着下次清理称为浮动垃圾

> **优缺点：**
>
> - **优点：**并发收集，低停顿
> - **缺点：**产生大量空间碎片，并发阶段会降低吞吐量

![GC - CMS](E:\Notes\JavaVirtualMachine.assets\GC - CMS.png)

### 6.4.4 G1(Garbage First)

> [Garbage-First (G1) Garbage Collector (oracle.com)](https://docs.oracle.com/en/java/javase/17/gctuning/available-collectors.html#GUID-C7B19628-27BA-4945-9004-EC0F08C76003)
>
> [garbage-first-g1-garbage-collector1](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-g1-garbage-collector1.html)
>
> > G1 is a mostly concurrent collector. Mostly concurrent collectors perform some expensive work concurrently to the application. This collector is designed to scale from small machines to large multiprocessor machines with a large amount of memory. It provides the capability to meet a pause-time goal with high probability, while achieving high throughput.
> >
> > G1 is selected by default on most hardware and operating system configurations, or can be explicitly enabled using `-XX:+UseG1GC` .
>
> 可以用于新老年代
>
> 回收过程：[Getting Started with the G1 Garbage Collector (oracle.com)](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)

1. **初始标记（Initial Marking）：** 标记以下GC Roots能够关联的对象，并且修改TAMS的值，需要暂停用户线程
2. **并发标记（Concurrent Marking）：** 从GC Roots进行可达性分析，找出存活的对象，与用户线程并发执行
3. **最终标记（Final Marking）：** 修正在并发标记阶段因为用户程序的并发执行导致变动的数据，需暂停用户线程
4. **筛选回收（Live Data Counting and Evacuation）：** 对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间制定回收计划

![GC - G1](E:\Notes\JavaVirtualMachine.assets\GC - G1.png)

**理解：**

1. 使用G1收集器时，Java堆的内存布局就与其它收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，他们都是一部分Region（不需要连续）的集合。

2. 每个Region大小都是一样的，可以是1M到32M之间的数值，但是必须保证是2的N次幂。

3. 如果对象太大，一个Region放不下（超过Region大小的50%），那么就会直接放到H中

   > Humongous objects are objects larger or equal the size of half a region. The current region size is determined ergonomically as described in the [Ergonomic Defaults for G1 GC](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-g1-garbage-collector1.html#GUID-082C967F-2DAC-4B59-8A81-0CEC6EEB9016) section, unless set using the option.`-XX:G1HeapRegionSize`

4. 设置Region大小：`‐XX:G1HeapRegionSize=M`

5. 所谓Garbage‐Frist，其实就是优先回收垃圾最多的Region区域

> The young generation contains eden regions (red) and survivor regions (red with "S"). These regions provide the same function as the respective contiguous spaces in other collectors, with the difference that in G1 these regions are typically laid out in a noncontiguous pattern in memory. Old regions (light blue) make up the old generation. Old generation regions may be humongous (light blue with "H") for objects that span multiple regions.
>
> An application always allocates into a young generation, that is, eden regions, with the exception of humongous objects that are directly allocated as belonging to the old generation.
>
> 年轻一代包含Eden区域（红色）和Survivor区域（红色带“S”）。这些区域提供与其他收集器中各自的连续空间相同的功能，不同之处在于，在 G1 中，这些区域通常在内存中以不连续的模式布局。旧区域（浅蓝色）组成了老一代。对于跨越多个区域的对象，旧一代区域可能是巨大的（浅蓝色带“H”）。
>
> 应用程序始终分配给年轻一代，即伊甸园区域，但直接分配给属于老一代的巨大对象除外。

![grbg_frst_hp](E:\Notes\JavaVirtualMachine.assets\grbg_frst_hp.png)

### 6.4.5 ZGC

> [z-garbage-collector](https://docs.oracle.com/en/java/javase/17/gctuning/z-garbage-collector.html)
> 
> > *The Z Garbage Collector (ZGC)* is a scalable low latency garbage collector. ZGC performs all expensive work concurrently, without stopping the execution of application threads for more than a few milliseconds. It is suitable for applications which require low latency. Pause times are independent of heap size that is being used. ZGC supports heap sizes from 8MB to 16TB.
>
> Java11引入的垃圾收集器
>
> 不管是物理上还是逻辑上，ZGC中已经不存在新老年代的概念了
>
> 会分为一个个page，当进行GC操作时会对page进行压缩，因此没有碎片问题
>
> 只能在64位的linux上使用，目前用得还比较少

- 可以达到10ms以内的停顿时间要求
- 支持TB级别的内存
- 堆内存变大后停顿时间还是在10ms以内

## 6.5 垃圾收集器分类

- 串行：Serial

  > 适合内存比较小的嵌入式设备

- 并行：Parallel

  > 更加关注吞吐量，适合科学计算、后台处理等弱交互场景

- 并发：CMS、G1

  > 更加关注停顿时间，适合WEB交互场景

![GC - Selector](E:\Notes\JavaVirtualMachine.assets\GC - Selector.png)

| 收集器                | 新老年代               | 串行 / 并行 / 并发 | STW                        | 指令                      |
| --------------------- | ---------------------- | ------------------ | -------------------------- | ------------------------- |
| **Serial**            | 新生代                 | 串行（单线程收集） | 暂停应用线程               |                           |
| **ParNew**            | 新生代                 | 并行（多线程收集） | 暂停应用线程               |                           |
| **Parallel Scavenge** | 新生代（吞吐优先）     | 并行（多线程收集） | 暂停应用线程               | `-XX:+UseParallelGC`      |
| **CMS**               | 老年代（停顿时间优先） | 并发               | 收集线程与用户线程一起工作 | `-XX:+UseConcMarkSweepGC` |
| **Serial Old**        | 老年代                 | 串行（单线程收集） | 暂停应用线程               |                           |
| **Parallel Old**      | 老年代（吞吐优先）     | 并行（多线程收集） | 暂停应用线程               | `-XX:+UseParallelGC`      |
| **G1**                | 新生代 / 老年代        | 并发               | 收集线程与用户线程一起工作 | `-XX:UseG1GC`             |

## 6.6 Reponsiveness and throughput

**停顿时间：**垃圾收集器进行垃圾回收终端应用执行响应的时间

**吞吐量：**运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)

## 6.7 什么时候会发生垃圾收集

> GC是由JVM自动完成的，根据JVM系统环境而定，所以时机是不确定的。
>
> 当然，我们是可以手动进行垃圾回收的，比如调用`System.gc()`方法通知JVM进行一次垃圾回收，但是具体什么时刻运行也无法控制。
>
> 也就是说`System.gc()`只是通知要回收，什么时候回收由JVM决定。但是不建议手动调用该方法，因为GC消耗的资源比较大。

1. 当Eden区或者S区不够用了：Young GC或者Minor GC
2. 老年代空间不够用了：Old GC或者Major GC
3. 方法区空间不够用了：Metaspace GC
4. `System.gc()`

> Full GC = Young GC + Old GC + Metaspace GC

# 7. Execution Engine

> User.java源码文件时Java这门高级开发语言，对程序员友好，方便我们开发
>
> javac编译器将User.java源码文件编译成class文件（这个编译成为前期编译），交给JVM运行，因为JVM只认识class字节码文件。同时在不同的操作系统上安装对应版本的JDK，里面包含了各自屏蔽操作系统底层细节的JVM，这样同一份class文件就能运行在不同的操作系统平台上，得益于JVM。这也是Write Once，Run Anywhere的原因所在。
>
> 最终JVM需要把字节码指令转换为机器码，可以理解为0101这样的机器语言，这样才能运行在不同的机器上，那么由字节码转变为机器码的操作，是由谁来完成操作的呢？这就是执行引擎里面的解释执行器和编译器所要做的事情。

![Execution Engine](E:\Notes\JavaVirtualMachine.assets\Execution Engine.png)

## 7.1 Interpreter

> 解释器逐条把字节码翻译成机器码并执行，跨平台的保证。
>
> 刚开始执行引擎只采用了解释执行的，但是后来发现某些方法或者代码块被调用执行的特别频繁时，就会把这些代码认定为“热点代码”。

## 7.2 即时编译器

> Just-In-Time compilation(JIT)，即时编译器先将字节码编译成对应平台的可执行文件，运行速度快。
>
> 即时编译器会把这些热点代码编译成与本地平台关联的机器码，并且进行各层次的优化，保存到内存中。

### 7.2.1 C1和C2

> HotSpot虚拟机里面内置了两个JIT：C1和C2
>
> > C1也称为Client Compiler，适用于执行时间短或者对启动性能有要求的程序
> >
> > C2也称为Server Compiler，适用于执行时间长或者对峰值性能有要求的程序
>
> Java7开始，HotSpot会使用分层编译的方式
>
> > 也就是会结合C1的启动性能优势和C2的峰值性能优势，热点方法会先被C1编译，然后热点方法中的热点会被C2再次编译

### 7.2.2 AOT

> 在Java9中，引入了AOT(Ahead of Time)编译器
>
> 即时编译器是在程序运行过程中，将字节码翻译成机器码。而AOT是在程序运行之前，将字节码转换为机器码
>
> > **优势 ：**这样不需要在运行过程中消耗计算机资源来进行即时编译
> >
> > **劣势 ：**AOT 编译无法得知程序运行时的信息，因此也无法进行基于类层次分析的完全虚方法内联，或者基于程序profifile的投机性优化（并非硬性限制，我们可以通过限制运行范围，或者利用上一次运行的程序profifile来绕开这两个限制）

### 7.2.3 Graal VM

> [GraalVM](https://www.graalvm.org/)
>
> 在Java10中，新的JIT编译器Graal被引入。它是一个以Java为主要编程语言，面向字节码的编译器。跟C++实现的C1和C2相比，模块化更加明显，也更加容易维护。
>
> Graal既可以作为动态编译器，在运行时编译热点方法；也可以作为静态编译器，实现AOT编译。
>
> 除此之外，它还移除了编程语言之间的边界，并且支持通过即时编译技术，将混杂了不同的编程语言的代码编译到同一段二进制码之中，从而实现不同语言之间的无缝切换。



> Spring Native：https://spring.io/blog/2021/03/11/announcing-spring-native-beta
>
> Dubbo Native：https://dubbo.apache.org/zh/docs/references/graalvm/support-graalvm/

# 8. 性能优化



