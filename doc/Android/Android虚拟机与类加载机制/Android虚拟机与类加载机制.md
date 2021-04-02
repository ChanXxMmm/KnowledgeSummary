- [1.JVM与Dalvik](#JVM与Dalvik)
- [2.ART与Dalvik](#ART与Dalvik)
- [3.ClassLoader](#ClassLoader)


# JVM与Dalvik
Android应用程序运行在Dalvik/ART虚拟机，并且每一个应用程序对应有一个单独的Dalvik虚拟机实例，Dalvik虚拟机实则也算是一个Java虚拟机，只不过它执行的不是class文件，而是dex文件

Dalvik虚拟机与Java虚拟机共享有差不多的特性，差别在于两者执行的指令时不一样的，前者的指令集是基于寄存器的，后者的指令时基于堆栈的，而寄存器的虚拟机与JVM版相比，指令数明显减少，数据移动次数也明显减少(相当于把局部变量表和操作数栈合并了)


# ART与Dalvik
Dalvik虚拟机执行的是dex字节码，解释执行。从Android 2.2版本开始，支持JIT即时编译（Just In Time）在程序运行的过程中进行选择热点代码（经常执行的代码）进行编译或者优化。

而ART（Android Runtime）是在Android4.4中引入的一个开发者选项，也是Android5.0及更高版本的默认Android运行时。ART虚拟机执行的是本地机器码。Android的运行时从Dalvik虚拟机替换成ART虚拟机，并不要求开发者将自己的应用直接编译成目标机器码，APK仍然是一个包含dex字节码的文件。

Dalvik下应用在安装的过程，会执行一次优化，将dex字节码进行优化生成odex文件。而Art下将应用的dex字节码翻译成本地机器码的最恰当AOT时机也就发生在应用安装的时候。ART引入了预先编译机制（Ahead Of Time），在安装时，ART使用设备自带的dex2oat工具来编译应用，dex中的字节码将被编译成本地机器码，所以我们在5.0，6.0的设备上进行安装的时候会比较慢

而到了Android N，ART使用预先 (AOT) 编译，并且从Android N混合使用AOT编译，解释和JIT
1. 最初安装应用时不进行任何 AOT 编译（安装又快了），运行过程中解释执行，对经常执行的方法进行JIT，经过 JIT 编译的方法将会记录到Profile配置文件中。
2. 当设备闲置和充电时，编译守护进程会运行，根据Profile文件对常用代码进行 AOT 编译。待下次运行时直接使用。

# ClassLoader

ClassLoader是一个抽象类，我们先来说下ClassLoader的继承关系

* ClassLoader
  * BaseDexClassLoader
    * PathClassLoader: Android应用程序类加载器
    * DexClassLoader: 额外提供的动态类加载器
  * BootClassLoader: 用于加载Android Framework层class文件

而类加载器是如何加载类的呢？我们进入ClassLoader的loadClass方法
```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
            // 看看是否已经加载过(由于要从文件读取，所以耗时，第一次要从文件读到缓存中)
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    //由于我们都是从PathClassLoader开始，而PathClassLoader的parent就是BootClassLoader
                    if (parent != null) {
                        //BootClassLoader
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            return c;
}
```
