- [1.泛型的定义](#泛型的定义)
- [2.我们为什么要使用泛型？](#我们为什么要使用泛型)
- [3.泛型的好处](#泛型的好处)
- [4.泛型类和泛型接口](#泛型类和泛型接口)
- [5.泛型方法](#泛型方法)
- [6.限定类型变量](#限定类型变量)
- [7.泛型中的约束和局限性](#泛型中的约束和局限性)

# 泛型的定义
参数化类型

# 我们为什么要使用泛型
以下两种情况就很好的说明了：

* 第一种情况：

    我们平时要使用两个数值类型求和操作时，需要
    ```java
    int addInt(int x,int y){
       return x+y;
    }
    ```
    但是如果此时增加了个需求，要用浮点类型进行求和操作，我们需要
    ```java
    float addFloat(float x,float y){
       return x+y;
    }
    ```
    假如此时又要增加double类型的，难道我们还要继续写一个addDouble方法吗？同一段代码，只是传入的参数不同，内部逻辑相同，就要重写一遍？这种情况下我们有没有什么方法写出一段代码，程序逻辑一样，根据     传入参数的不同自动进行计算呢？

* 第二种情况
    ```java
    List list = new ArrayList();
    list.add("list");
    list.add(100);

    for (int i = 0; i < list.size(); i++) {
        System.out.println("list:"+(String) list.get(i));
    }
    ```
    编译阶段没有任何问题，我们执行下看看：
    ```java
    java.lang.ClassCastException:java.lang.Integer cannot be cast to java.lang.String at...
    ```
    类型转换异常，由于List没有定义泛型类型，此时的List的类型就是Object，所以我们在存储时没有问题，但当我们去取数据时，就要人为的去强制转换具体的目标类型。

# 泛型的好处
根据上面的两种情况：

1. 适用于多种数据类型执行相同的代码。
2. 泛型中的类型在使用时指定，不需要强制转换(在编译阶段就能指定数据类型，插入错误数据类型立即发现，避免了使用时强制转换)。

# 泛型类和泛型接口
* 泛型类：
```java
public class Test<T> {
    private T data;
    
    public Test() {}
    
    //构造函数中可以使用
    public Test(T data) {
        this.data = data;
    }

    //get中可以使用
    public T getData() {
        return data;
    }

    //set中可以使用
    public void setData(T data) {
        this.data = data;
    }

    public static void main(String[] args) {
        Test<String> test = new Test<>();
        
        //由于我们定义的Test泛型为String，所以我们在使用时必须是String类型
        test.setData("test");
        
        System.out.println(test.getData());

    }
}
```

* 泛型接口：
```java
public interface Test<T>{
    public T next();
}

//第一种实现
public class TestImpl<T> implements Test<T>{
  @Override
    public T next() {
        return null;
    }
}

//第二种使用
public class TestImpl implements Test<String>{
  @Override
    public String next() {
        return null;
    }
}
```
# 泛型方法
```java
public class Test{
    public <T> T getMethod(T...a){
        return a[a.length/2];
    }

    public static void main(String[] args) {
        Test test = new Test();
        
        //没有定义泛型类型
        System.out.println(test.getMethod(1,2,3));

        //定义泛型类型为String
        System.out.println(test.<String>getMethod("1","2","3"));
    }
}
```
# 注意：
1. 无论是泛型方法，方法类，还是泛型接口都是<T>来定义泛型，尖括号 <>中的 T 被称作是类型参数，用于指代任何类型。

    比如：

  *  UnKnown class 'E'
  ```java  
  public class Test<K>{
    //E没有定义，肯定会报错"UnKnown class 'E'"
    public E getMethod(E key){
        return key;
    }
  }
  ```
  * 形参
  
  ```java
  public class Test<T>{
    //不是泛型方法，只是一个普通方法，使用了Test<Number>作为形参而已
    public void getMethod(Test<Number> a){}  
  }
  ```
  * 其他 
  
  ```java
public class MyClass {
    static class Fruit{
        @Override
        public String toString() {
            return "fruit";
        }
    }

    static class Apple extends Fruit{
        @Override
        public String toString() {
            return "apple";
        }
    }

    static class Person{
        @Override
        public String toString() {
            return "person";
        }
    }

    static class Test<T>{
    
        //使用了泛型类定一个泛型
        public void show(T t){
            System.out.println(t.toString());
        }
        
        //定义了一个新的泛型E
        public <E> void show_2(E t){
            System.out.println(t.toString());
        }
        
        //定义了一个新的泛型T，与泛型类定一个T不同，不是同一个泛型
        public <T> void show_3(T t){
            System.out.println(t.toString());
        }
    }

    public static void main(String[] args) {
        Apple apple = new Apple();
        Person person = new Person();
        
        //由于Apple是Fruit的子类，所以可以打印
        Test<Fruit> test = new Test();
        test.show(apple);
        
        //编译器会报错
        test.show(person);
        
        //由于show_2方法本身定义了一个新的泛型，所以可以打印
        test.show_2(person);
        
        //由于show_3方法本身定义了一个新的泛型，所以可以打印
        test.show_3(person);
    }
}
```

2. 同时不光可以定义一个，<K,V>可也是可以的。
  ```java
  //定义多个泛型
  public class Test<K,V>{}
  ```

# 限定类型变量

先看下面代码

```java
public static <T> T min(T a,T b){
        if (a.compareTo(b)>0) return a; else return b;
}
```
此段代码有问题：由于定义的泛型T不知道是什么类型的，所以无法保证他有compareTo方法。

所以为了解决这个问题就有了类型变量的限定
```java
//只有继承的父类或者自己实现了Comparable接口才可以
public static <T extends Comparable> T min(T a,T b){
        if (a.compareTo(b)>0) return a; else return b;
}
```
写法上<T extends String>也支持多个泛型和extends多个接口或者类，比如<K extends String,V extends ArrayList&Comparable>,这时候类必须写在接口前面且类只能有一个，同时不光可以在泛型方法中使用，还可以在泛型类上使用
    
# 泛型中的约束和局限性
* 不能实例化类型变量
```java
public class Test<T>{
    public void get(){
        //不允许
        T t = new T();
    }
}
```
* 静态域或者静态方法里不能饮用类型变量
```java
public class Test<T>{
    //不允许，因为在new Test的时候，先执行静态的，之后才是Test构造方法，此时T还没有确定，所以无法在静态域或者静态方法中使用
    private static T instance;
}
```
    但如果静态方法本身是泛型方法就可以
```java
public class Test<T>{

    private static <T> T get(){
        return null;
    }
}
```
* 不能用基本类型实例化类型参数
```java
public class Test<T>{
    public static void main(String[] args) {
        //泛型中所有的基本数据类型都不可以，因为它不是个对象
        Test<double> t = new Test();
        
        //只能使用它们的包装类
        Test<Double> t = new Test();
    }
}




    
