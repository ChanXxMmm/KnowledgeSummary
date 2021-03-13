- [1.注解的作用或者定义](#注解的作用或者定义)
- [2.自定义注解](#自定义注解)
- [3.元注解](#元注解)
- [4.注解的使用场景](#注解的使用场景)

# 注解的作用或者定义

注解本身没有任何意义，单独的注解就是一种注解，他需要结合其他如反射，插庄等技术才有意义。

# 自定义注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
//上面都是元注解，接下来会介绍
//自定义注解@interface + 注解名 + 元素(可以没有)
public @interface MyAnnotation {
    String value();
    String data() default "xxx";
}

//注解有元素，则需要传参，除非注解元素有default
@MyAnnotation("test")
public class Test{
}
```
value()是默认的key，也就是说他可以省略掉key，但如果是多个参数则value不能省略
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface MyAnnotation {
    String value();
    int id();
    String data() default "xxx";
}

@MyAnnotation(value="test",id = 1)
public class Test{
}
```

# 元注解

元注解：注解上的注解

* @Target:定义注解所作用的位置(类，方法，属性等)，默认是全部,可以多个@Target(ElementType.TYPE,ElementType.METHOD)
```java
@Target(ElementType.TYPE)   //接口、类、枚举
@Target(ElementType.FIELD) //字段、枚举的常量
@Target(ElementType.METHOD) //方法
@Target(ElementType.PARAMETER) //方法参数
@Target(ElementType.CONSTRUCTOR)  //构造函数
@Target(ElementType.LOCAL_VARIABLE)//局部变量
@Target(ElementType.ANNOTATION_TYPE)//注解
@Target(ElementType.PACKAGE) ///包   
```
* @Retention:定义注解所保留的级别，SOURCE<CLASS<RUNTIME
```java
@Retention(RetentionPolicy.SOURCE)  //标记的注解仅保留在源码级别，冰杯编译器忽略
@Retention(RetentionPolicy.CLASS)  //标记的注解在编译时由编译器保留，但Java虚拟机忽略
@Retention(RetentionPolicy.RUNTIME)  //标记的注解由JVM保留，因此运行时环境可以使用它
```

# 注解的使用场景

根据注解的保留级别不同，对注解的使用自然存在不同的场景


 | 保留级别 | 使用技术 | 说明
 | :---: |:---: | ---
 |源码| APT | 在编译期能够获取注解与注解声明的类包括类中所有成员信息，一般用于生成额外的辅助类。
 |字节码 | 字节码增强 | 在编译出Class后，通过修改Class数据以实现修改代码逻辑目的。对于是否需要修改的区分或者修改为不同逻辑的判断可以使用注解。
 |运行时 | 反射 | 在程序运行期间，通过反射技术动态获取注解与其元素，从而完成不同的逻辑判定。



