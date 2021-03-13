- [1.什么是反射](#什么是反射)
- [2.获取类的对象](#获取类的对象)
- [3.根据类得到类名](#根据类得到类名)
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
# 根据类得到类名

1. getName() 全限定名
```java
打印: HashMap.class.getName()
输出: java.util.HashMap
```
2. getSimpleName() 类名
```java
打印: HashMap.class.getSimpleName()
输出: HashMap
```
3. getPackage() 包名
```java
打印: HashMap.class.getPackage()
输出: package java.util
```

# Field类
1. getField("属性名") 获取公共属性
```java
public class Test{
      public int index = 100;
}
打印: Test.class.getField("index")
输出: 包名.Test.index
```
2. getName() 属性名
```java
public class Test{
      public int index = 100;
}
打印: Test.class.getField("index").getName()
输出: index
```
3. getModifiers() 修饰符

 | 默认 | public | private | protected | static |  final |
 | :---: |:---: | :---: |:---: |:---: |:---: |
 |0| 1 | 2 | 4 | 8 | 16
```java
public class Test{
      public int index = 100;
}
打印: Test.class.getField("index").getModifiers()
输出: 1
```
4.getType()数据类型
```java
public class Test{
      public int index = 100;
}
打印: Test.class.getField("index").getType()
输出: int
```
5.set(对象名，属性值) 
```java
public class Test{
      public int index = 100;
}
Test test = new Test();
Class testClass = test.getClass();
testClass.getField("index").set(test,20);
打印: test.index
输出: 20
```

6.get(对象名)
```java
public class Test{
      public int index = 100;
}
Test test = new Test();
Class testClass = test.getClass();
Field index = testClass.getField("index");
index.set(test,20);
打印: index.get(test)
输出: 20
```

7.getDeclaredField("属性名") 获取属性
```java
public class Test{
      public int index = 100;
}
打印: Test.class.getDeclaredField("index")
输出: public int 包名.Test.index
```
8.setAccessible(true) 设置私有能访问
```java
public class Test{
      private int index = 100;
}
Test test = new Test();
Class testClass = test.getClass();
Field index = testClass.getField("index");
index.setAccessible(true);
index.set(test,20);
打印: index.get(test)
输出: 20
```
9.getDeclaredFields() 所有属性
```java
public class Test{
      public String id;
      private int index = 100;
}
Field[] declaredFields = Test.class.getDeclaredFields();
打印: 遍历declaredFields
输出:
public java.lang.String 包名.Test.id
private int 包名.Test.index
```
