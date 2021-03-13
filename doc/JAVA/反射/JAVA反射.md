- [1.什么是反射](#什么是反射)
- [2.获取类的对象](#获取类的对象)
- [3.根据类得到类名](#根据类得到类名)
- [4.Field类](#Field类)
- [5.Method类](#Method类)
- [6.构造方法](#构造方法)
- [7.实操之通过注解和反射实现findViewById](#实操之通过注解和反射实现findViewById)


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
1. getField("属性名") 获取公共属性(自己+父类)
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

7.getDeclaredField("属性名") 获取属性(只有自己的，不包含父类)
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

# Method类
1. getMethod(方法名，参数数据类型(无参数传null)) 获取公共方法
```java
public class Test{
    public void doSth(int index){}
}
打印: Test.class.getMethod("doSth", int.class)
输出: public void 包名.Test.doSth(int)
```
2. getDeclaredMethod(方法名，参数数据类型(无参数传null)) 获取私有方法
```java
public class Test{
    private void doSth(int index){}
}
打印: Test.class.getDeclaredMethod("doSth", int.class)
输出: private void 包名.Test.doSth(int)
```
3. invoke(对象名，参数列表(理解为方法名)) 执行方法
```java
public class Test{
   private void doSth(int index){
       System.out.println("doSth:"+index);
   }
}
Test test = new Test();
Method method = test.getClass().getDeclaredMethod("doSth", int.class);
method.setAccessible(true);
method.invoke(test,23);
输出: doSth:23
```
4. getParameterTypes() 得到参数列表
```java
public class Test{
      private void doSth(int index,String id){}
}
Method method = Test.class.getDeclaredMethod("doSth", int.class, String.class);
Class<?>[] parameterTypes = method.getParameterTypes();
打印: 遍历parameterTypes
输出:
int
class java.lang.String
```

5. getDecaledMethods() 得到类的所有方法
```java
public class Test{
    private void doSth(){}
    public void doSthElse(int i){}
}
Method[] declaredMethods = Test.class.getDeclaredMethods();
打印: 遍历declaredMethods
输出:
private void 包名.Test.doSth()
public void 包名.Test.doSthElse(int)
```
6.  getReturnType() 得到方法返回值的数据类型
```java
public class Test{
      private void doSth(int index,String id){}
}
Method method = Test.class.getDeclaredMethod("doSth", int.class, String.class);
打印: method.getReturnType()
输出: void
```
# 构造方法
1. Class对象.getConstructor()得到构造方法
```java
public class Test{
     public Test(){}
     public Test(String id){}
}
打印: Test.class.getConstructor()
输出: public 包名.Test()
```

2. Class对象.getConstructors()得到所有构造方法
```java
public class Test{
     public Test(){}
     public Test(String id){}
}
打印: 遍历Test.class.getConstructors()
输出: 
public 包名.Test()
public 包名.Test(java.lang.String)
```

# 实操之通过注解和反射实现findViewById

   初始代码:
      ```java
      public class MainActivity extends AppCompatActivity {
          int i;
          int m;
          private TextView textView;
          @Override
          protected void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              setContentView(R.layout.activity_main);
          }
      }
      ```
   可以看到就是一个很简单的界面，有一个TextView，id为tv,现在我们要做的就是通过注解和反射自动实现findViewById
   
   现在要先做一个工具类
   1. 可以获取到MainActivity中所有的属性
      ```java
      public class InjectUtils {
        public static void injectView(Activity activity){
              //通过传进来的Activity，获得其Class对象
              Class<? extends Activity> aClass = activity.getClass();
              //通过Class对象获取它的所有属性
              Field[] fields = aClass.getDeclaredFields();
              //遍历所有属性通过区分得到TextView
              for (Field field: fields) {
                  
              }
       }
      }
      
      ```
   2. 由于此时无法区分哪个是TextView，所以此时要添加我们自己的注解来区分，首先生成一个自己的注解用来接收View的id
         ```java
         @Target(ElementType.FIELD)//该注解只会作用在属性上，也就是我们的view上
         @Retention(RetentionPolicy.RUNTIME)//只会在运行期间自动实现findViewById
         public @interface InjectView {
            //通过int接收view的id，同时使用@IdRes注解来进行限制，只能输入R.id.xxx
            @IdRes int value();
         }
         ```
         
   3. 此时给我们需要自动findViewById的控件加上注解
      ```java
      @InjectView(R.id.tv)
      private TextView textView;
      
      ```
   4. 此时可以继续补全自动实现工具类
      ```java
      public class InjectUtils {
        public static void injectView(Activity activity){
              //通过传进来的Activity，获得其Class对象
              Class<? extends Activity> aClass = activity.getClass();
              //通过Class对象获取它的所有属性
              Field[] fields = aClass.getDeclaredFields();
              //遍历所有属性通过区分得到TextView
              for (Field field: fields) {
                  if (field.isAnnotationPresent(InjectView.class)){
                  
                      //找到使用我们注解的属性，也就是控件
                      InjectView annotation = field.getAnnotation(InjectView.class);
                      
                      //这时候就可以去获取注解参数拿到控件的id
                      int id = annotation.value();
                      
                      //拿到id后通过findViewById拿到View
                      View view = activity.findViewById(id);

                      //通过反射给到控件的值
                      field.setAccessible(true);
                      try {
                          field.set(activity,view);
                      } catch (IllegalAccessException e) {
                          e.printStackTrace();
                      }
                  }
              }
         }
       }
      
       ```
  5. 使用
  ```java
  public class MainActivity extends AppCompatActivity {
    @InjectView(R.id.tv)
    TextView textView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //使用我们的工具自动实现findViewById
        InjectUtils.injectView(this);
        
        textView.setText("Hello");
    }
  }
  ```


