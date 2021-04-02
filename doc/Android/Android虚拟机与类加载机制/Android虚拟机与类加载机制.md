- [1.JVM与Dalvik](#JVM与Dalvik)
- [2.ART与Dalvik](#ART与Dalvik)

# JVM与Dalvik
Android应用程序运行在Dalvik/ART虚拟机，并且每一个应用程序对应有一个单独的Dalvik虚拟机实例，Dalvik虚拟机实则也算是一个Java虚拟机，只不过它执行的不是class文件，而是dex文件

Dalvik虚拟机与Java虚拟机共享有差不多的特性，差别在于两者执行的指令时不一样的，前者的指令集是基于寄存器的，后者的指令时基于堆栈的，而寄存器的虚拟机与JVM版相比，指令数明显减少，数据移动次数也明显减少(相当于把局部变量表和操作数栈合并了)


# ART与Dalvik


