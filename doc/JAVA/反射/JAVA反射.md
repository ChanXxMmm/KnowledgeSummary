- [1.什么是反射](#什么是反射)
- [2.获取类的对象](#获取类的对象)
- [3.根据类得到类名(全限定名)](#根据类得到类名(全限定名))
- [4.Field类](#Field类)
- [5.Method类](#Method类)
- [6.构造方法](#构造方法)


# 什么是反射

反射就是在运行状态中,对于任意一个类,都能够知道这个类的所有属性和方法;对于任意一个对象,都能够调用它的任意方法和属性;并且能改变它的属性。是Java被视为动态语言的关键。

# 获取类的对象
1. 类名.Class

```java
打印: HashMap.class
输出: class java.util.HashMap
```
2. 对象.getClass()
```java
打印: new HashMap<>().getClass()
输出: class java.util.HashMap
```
3. Class.forName("全限定名")
```java
打印: Class.forName("java.util.HashMap")
输出: class java.util.HashMap
```
4. 类.getClassLoader.loadClass("全限定名")
```java
打印: getClassLoader().loadClass("java.util.HashMap")
输出: class java.util.HashMap
```
5. 子类.class.getSuperClass()
```java
static class Test extends HashMap{}
打印: Test.class.getSuperclass()
输出: class java.util.HashMap
```
6. 包装类.class
```java
打印: Boolean.class
输出: class java.lang.Boolean
```
# 根据类得到类名(全限定名)

1. getName()全限定名
```java
打印: HashMap.class.getName()
输出: java.util.HashMap
```
2. getSimpleName()类名
```java
打印: HashMap.class.getSimpleName()
输出: HashMap
```
3. getPackage()包名
```java
打印: HashMap.class.getPackage()
输出: package java.util
```

# Field类
