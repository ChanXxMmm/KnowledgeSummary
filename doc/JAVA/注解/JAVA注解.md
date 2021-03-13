- [1.注解的作用或者定义](#注解的作用或者定义)
- [2.元注解](#元注解)
- [2.自定义注解](#自定义注解)


# 注解的作用或者定义

注解本身没有任何意义，单独的注解就是一种注解，他需要结合其他如反射，插庄等技术才有意义。

# 元注解

元注解：注解上的注解
```java
@Target：定义注解所作用的位置(类，方法，属性等)，默认是全部

@Target(ElementType.TYPE)   //接口、类、枚举
@Target(ElementType.FIELD) //字段、枚举的常量
@Target(ElementType.METHOD) //方法
@Target(ElementType.PARAMETER) //方法参数
@Target(ElementType.CONSTRUCTOR)  //构造函数
@Target(ElementType.LOCAL_VARIABLE)//局部变量
@Target(ElementType.ANNOTATION_TYPE)//注解
@Target(ElementType.PACKAGE) ///包   
```


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

