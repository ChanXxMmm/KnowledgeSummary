- [1.JVM与操作系统的关系](#JVM与操作系统的关系)
- [2.JVM运行过程](#JVM运行过程)
- [3.运行时数据区](#运行时数据区)
  - [程序计数器](#程序计数器)
  - [虚拟机栈](#虚拟机栈)
  - [本地方法栈](#本地方法栈)
  - [方法区](#方法区)
  - [Java堆](#Java堆)
  - [从底层深入理解运行时数据区](#从底层深入理解运行时数据区)

# JVM与操作系统的关系
* 什么是JVM

  JVM -- Java Virtual Machine(Java虚拟机)

* JVM的作用

  JVM的作用就是翻译: 我们平时写的代码都是Java程序，通常是.java后缀，通过javac编译成.class文件，也就是Java字节码，而JVM可以识别字节码的同时还要调操作系统的一些函数，说直白点就是JVM能将字节码翻译成操作系统能够识别的机器码
  
* 跨平台到跨语言
  
  比如我们在接触到的第一个程序HelloWorld.java，它通过javac编译成.class或者打包成jar等，而它在不同操作系统上运行的效果都是一样的，这就是跨平台。
  
  而随着Kotlin的出现，JVM也能够识别Kotlin语言，将它生成的字节码翻译成操作系统能够识别的，当然还有很多其他语言是一样的，这就是跨语言。
  
扩展: JavaSe体系架构
1. JVM只是一个翻译
2. JRE提供了基础类库: 比如io，网络等
3. JDK提供了工具: 比如上面说的将javac工具,反编译用的javap工具等

# JVM运行过程

![image](https://user-images.githubusercontent.com/61224872/111259295-954d1400-8659-11eb-8724-03f6851775eb.png)

* 首先我们编写一个HelloWord.java文件
* 通过javac工具将上述文件编程成HelloWord.class文件，就是上面所说的字节码
* 通过JVM
  * 首先要做的就是通过Java类加载CLassLoader加载到运行时数据区(它就是JVM所管理的内存)中
  * 之后通过JVM中的执行引擎去将运行时数据区中的数据解释执行或者JIT执行操作系统(解释执行可以理解为翻译一行执行一行，JIT可以理解为热点数据，比如一个方法翻译执行了100次，1000次，它就会认为是热点数据，一直是解释执行没有意义，效率很低，它会直接编译成本地代码，提高运行速度)

# 运行时数据区

* 定义:

  Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域

![image](https://user-images.githubusercontent.com/61224872/111259752-8d41a400-865a-11eb-9061-6459d1b33363.png)

我们可以看到运行时数据分可以划分为线程共享和线程隔离/私有的由于多线程的存在

线程私有: 比如三个线程，那么在运行时数据区中，就会有三份线程私有的数据区，分别包含虚拟机栈，本地方法栈，程序计数器。

线程共享: 不受线程影响的有方法区和堆

下面来详细介绍下运行时数据区中的东西:
# 程序计数器
  * 在多线程的环境下，程序计数器负责指向当前线程正在执行的字节码指令的地址
  * 为了方便理解，我们直接上代码
  ```java
  public class Test {
    public Test() {
    }

    public int work() {
        int x = 1;
        int y = 2;
        int z = (x + y) * 10;
        return z;
    }

    public static void main(String[] args) throws Exception {
        Test test = new Test();
        test.work();
    }
  }
  ```
    很简单的一段代码，我们编译下Test类，会在build中生成一个Test.class文件，我们通过javap -c Test.class来看看他的字节码汇编后长什么样:
  ```java
  //我们直接看work方法的样子
  public int work();
    Code:
       0: iconst_1
       1: istore_1
       2: iconst_2
       3: istore_2
       4: iload_1
       5: iload_2
       6: iadd
       7: bipush        10   //大部分指令的偏移量是1，但是如果偏移量过大，就不会是1，比如此时跳过了8
       9: imul
      10: istore_3
      11: iload_3
      12: ireturn
  ```
    我们可以看到我们的程序变成字节码后，有一个Code行号，会按照Code行号从0~12一行一行执行后面的指令，Code表示针对work方法体的偏移量，大体上可以理解为程序计数器记录的地址
  * 可能有人会问了，为什么需要程序计数器？
    因为操作系统的时间轮转机制，程序计数器是针对当前线程而言，如果时间轮转机制将当前线程挂起，当线程再次获得时间片后，程序计数器就保证了下次该执行哪一条指令
  * 程序计数器会OOM吗？
    在JVM内存区域中，程序计数器是唯一不会OOM的，因为它是一块很小的内存区域，它只负责记录刚才我们所说的地址，一般用一个int类型的记录就够了
    
# 虚拟机栈
 
  ![image](https://user-images.githubusercontent.com/61224872/111265527-2fb25500-8664-11eb-996c-35c0a82e8937.png)

  
  * 栈是什么样的数据结构？ FIFO先进后出
  * 虚拟机栈在JVM运行过程中存储当前线程运行方法所需的数据，指令、返回地址
  * 虚拟机栈内部包含栈帧
    * 局部变量表: 存储方法的局部变量(八大数据类型和引用)，比如上面work方法中的x,y,z，比如main方法中的person引用(new Person是对象，Person person中的person就是引用，指向了前面的对象)
    * 操作数栈: 存储方法的操作/执行，我们还是以代码为例
    ```java
     public int work() {
        int x = 1;
        int y = 2;
        int z = (x + y) * 10;
        return z;
    }
    
    public int work();
    Code:
       0: iconst_1           //将int型1入操作数栈，可以理解为创建一个int型变量数值为1加入到操作数栈中
       1: istore_1           //将操作数栈中栈顶int型数值存入到局部变量表下标为1的位置，可以理解为将刚才加入的变量从操作数栈移到局部变量表下标为1的位置(由于不是静态方法，0的位置都是this)，等于上面两步的操作就是 int x = 1;
       2: iconst_2           //将int型2入操作数栈
       3: istore_2           //将操作数栈中栈顶int型数值存入到局部变量表下标为2的位置，等于上面两步的操作就是 int y = 2;
       4: iload_1            //将局部变量表下标为1的数据入操作数栈
       5: iload_2            //将局部变量表下标为2的数据入操作数栈
       6: iadd               //将操作数栈中栈顶两int类型值出栈，相加，并将结果压入操作数栈，等于上面三步完成了 x+y
       7: bipush        10   //将10扩展成int入操作数栈
       9: imul               //将操作数栈栈顶两int数值出栈，相乘，并将结果压入操作数栈，等于完成了(x + y) * 10
      10: istore_3           //将操作数栈中栈顶int型数值存入到局部变量表下标为3的位置
      11: iload_3            //将局部变量表下标为3的数据入操作数栈
      12: ireturn
    ```
    我们可以看到，平时我们所说的栈操作，指的不是虚拟机栈，而是它内部栈帧的操作数栈
    
    * 动态链接: Java语言特性多态，需要类运行时才能确定具体的方法
    ```java
    //比如刚才的Person类如果是个抽象类，他有两个子类Man和Women
    Person tom = new Man();
    tom.work();
    tom = new Woman();
    tom.work();
    ```
    我们可以看到，上端代码在编译器没有任何问题，但是tom无法确定指向Man还是Woman，所在运行期间，就需要动态链接来判断到底是Man还是Woman
    
    * 完成出口: 可以理解为返回地址，比如上面的demo中说的work方法执行完毕，就会打包成一个栈帧携带者z值出栈，此时就会有一个返回地址，也就是方法出口。而具体地址如果方法正常返回则使用程序计数器中的地址作为返回地址，如果异常则要去异常处理表来进行确定
    
# 本地方法栈
本地方法栈保存的是native方法的信息

比如我们的对象都是Object，它都有一个hashCode方法
```java
public native int hashCode();
```
它都会调用底层native方法(内部调用C语言)，当一个JVM创建的线程调用native方法后，JVM不再为其在虚拟机栈中创建栈帧，JVM只是简单地动态链接并直接调用native方法

# 方法区
方法区中存放了类信息，常量，静态变量，即时编译期编译后的代码

* 扩展
1. JDK1.7之前方法区的实现叫永久代，它的划分会跟堆有关系，先根据堆来划分新生代和老年代，再以这样的规范划分了永久代(同时受限于堆的大小)，但是发现JVM不光回收堆，还要回收永久代，但是永久代都是很难回收的，导致回收效率低，所以在JDK1.8之后永久代变为了元空间
2. JDK1.8之后方法区的实现叫元空间，可以直接使用机器内存(直接内存，堆外内存，它不是JVM运行时数据区的一部分，基本是堆中创建一个对象直接引用直接内存)，不再受堆得大小限制,所以它的好处就是方便拓展，但是坏处是会挤压堆大小
3. 任意版本都叫方法区

* 类信息: 就是我们之前所说的Helloworld.class通过类加载器加载到运行时数据区，加载的就是这个类，到方法区中

# Java堆
Java堆中存储着对象实例和数组

可能有人会问为什么方法区和堆中存储的都是共享的数据，为什么要分开呢？因为堆中存储的对象实例和数组都是要被频繁回收的，所以保持不变的和回收难度大的放在了方法区，经常创建和回收的放在了堆，便于垃圾回收的高效

# 从底层深入理解运行时数据区
我们先来看一段代码
```java
public class JVMObject {
    public final static String MAN_TYPE = "man"; 
    public static String WOMAN_TYPE = "woman";  

    public static void  main(String[] args)throws Exception {
        Teacher T1 = new Teacher();
        T1.setName("Mark");
        T1.setSexType(MAN_TYPE);
        T1.setAge(36);
        for (int i=0;i<15;i++){//进行15次垃圾回收
            System.gc();//垃圾回收
        }
        Teacher T2 = new Teacher();
        T2.setName("King");
        T2.setSexType(MAN_TYPE);
        T2.setAge(18);
        Thread.sleep(Integer.MAX_VALUE);//线程休眠很久很久
    }
}

class Teacher{
    String name;
    String sexType;
    int age;

    ......//省略set和get
}
```
上述代码在执行时是如何一步一步的通过JVM进入到运行时数据区的呢？

1. 首先申请内存: 栈内存，堆内存，方法区内存
2. 通过类加载器加载JVMObject.class和Teacher.class，加载到运行时数据区的方法区
3. 方法区加载类后进行拆分，将MAN_TYPE常量和WOMAN_TYPE静态变量放到方法区
4. main方法跑起来后，在虚拟机栈中创建main进程的虚拟机栈，同时在该虚拟机栈中压入main方法的栈帧
5. 执行Teacher T1 = new Teacher()，会在堆Eden区中创建一个T1的Teacher对象，同时在栈帧中的局部变量表中有一个T1的引用
6. 执行set方法: 在栈帧中的操作数栈中不断的入栈出栈
7. 执行15次gc操作，此时垃圾回收器主要对堆进行回收，堆中的T1经历过15次GC后进入到老年代，为什么是15次？在后面的垃圾回收机制中会介绍
8. 执行Teacher T2 = new Teacher()，会在堆Eden区中创建一个T2的Teacher对象，同时在栈帧中的局部变量表中有一个T2的引用
9. 执行set方法: 在栈帧中的操作数栈中不断的入栈出栈

此时JVM的运行时数据区的效果如下图
![image](https://user-images.githubusercontent.com/61224872/111302764-bda33580-868e-11eb-9b12-6b65a8bbaf59.png)





