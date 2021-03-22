- [1.什么是反射](#什么是反射)
- [2.获取类的对象](#获取类的对象)
- [3.根据类得到类名](#根据类得到类名)
- [4.Field类](#Field类)
- [5.Method类](#Method类)
- [6.构造方法](#构造方法)
- [7.反射获取泛型真实类型](#反射获取泛型真实类型)
- [8.实操之通过注解和反射实现findViewById](#实操之通过注解和反射实现findViewById)
- [9.实操之通过注解和反射实现getIntent](#实操之通过注解和反射实现getIntent)
- [10.实操之通过注解反射和动态代理实现onClick](#实操之通过注解反射和动态代理实现onClick)

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
# 反射获取泛型真实类型

当我们对一个泛型类进行反射时，需要得到泛型中的真实数据类型，此时需要通过Type体系来完成，它包含一个实现类和四个接口:

1. 实现类就是Class

2. GenericArrayType接口:当需要描述的类型是泛型类的数组时，此接口会作为Type的实现。它的组成元素是ParameterizedType或TypeVariable类型
```java
public class TestType<T> {

    List<String>[] lists;

    public static void main(String[] args) throws Exception {
        Field f = TestType.class.getDeclaredField("lists");
        GenericArrayType genericType = (GenericArrayType) f.getGenericType();
    }
}
打印: genericType.getGenericComponentType()  //返回数组的组成对象
输出: java.util.List<java.lang.String>
```


3. ParameterizedType接口:具体的泛型类型，可以获得元数据中泛型签名类型(泛型真实类型)
```java
public class TestType {
    Map<String, String> map;

    public static void main(String[] args) throws Exception {
        Field f = TestType.class.getDeclaredField("map");
        ParameterizedType pType = (ParameterizedType) f.getGenericType();
    }
}
打印: pType.getRawType()  //返回承载该泛型信息的对象
输出: interface java.util.Map

打印: 遍历pType.getActualTypeArguments()  //返回实际泛型类型列表
输出: 打印两遍: class java.lang.String
```


4. TypeVariable接口:泛型类型变量，可以得到泛型上下限等信息(但是类型变量在定义的时候只能使用extends进行(多)边界限定, 不能用super)
```java
public class TestType <K extends Comparable & Serializable, V> {
    K key;
    V value;
    public static void main(String[] args) throws Exception {
        // 获取字段的类型
        Field fk = TestType.class.getDeclaredField("key");
        Field fv = TestType.class.getDeclaredField("value");

        TypeVariable keyType = (TypeVariable)fk.getGenericType();
        TypeVariable valueType = (TypeVariable)fv.getGenericType();
    }
}
打印: keyType.getName() 和 valueType.getName()  //获取泛型在源码中定义时的名字
输出: K 和 V

打印: keyType.getGenericDeclaration()  //获取声明该类型变量的类型
输出: class com.test.TestType

打印: 遍历keyType.getBounds() 和 遍历valueType.getBounds()  //获取类型变量的上边界
输出: 
class com.test.TestType 和 interface java.io.Serializable
class java.lang.Object // V没明确声明上界的, 默认上界是 Object
```

5. WildcardType接口:通配符泛型，获得上下线信息
```java
public class TestType {
    private List<? extends Number> a;  // 上限
    private List<? super String> b;     //下限

    public static void main(String[] args) throws Exception {
        Field fieldA = TestType.class.getDeclaredField("a");
        Field fieldB = TestType.class.getDeclaredField("b");
        // 先拿到范型类型
        ParameterizedType pTypeA = (ParameterizedType) fieldA.getGenericType();
        ParameterizedType pTypeB = (ParameterizedType) fieldB.getGenericType();
        // 再从范型里拿到通配符类型
        WildcardType wTypeA = (WildcardType) pTypeA.getActualTypeArguments()[0];
        WildcardType wTypeB = (WildcardType) pTypeB.getActualTypeArguments()[0];
    }
}
打印: wTypeA.getUpperBounds()[0]
输出: class java.lang.Number

打印: wTypeB.getLowerBounds()[0]
输出: class java.lang.String

打印: wTypeA
输出: ? extends java.lang.Number
```
6. 关于日常开发中泛型反射遇到的问题

在我们的实际开发过程中，经常会遇到访问服务器获取结果
```
static class Response<T> {
      T data;
      int code;
      String message;
      @Override
      public String toString() {
            return "Response{" +
                    "data=" + data +
                    ", code=" + code +
                    ", message='" + message + '\'' +
                    '}';
      }

      public Response(T data, int code, String message) {

          this.data = data;
          this.code = code;
          this.message = message;
      }
}
```
服务器返回的Response中除了状态码code和对应的Message，还有数据，但是数据有多种多样的类型，所以用泛型定义。

此时比如有一个Data实体来作为Response中的data
```
static class Data {
        String result;

        public Data(String result) {
            this.result = result;
        }

        @Override
        public String toString() {
            return "Data{" +
                    "result=" + result +
                    '}';
        }
}

Response<Data> dataResponse = new Response(new Data("数据"), 1, "成功");
Gson gson = new Gson();
String json = gson.toJson(dataResponse);

//反序列化
Response<Data> response = gson.fromJson(json, Response.class);

打印: json
输出: {"data":{"result":"数据"},"code":1,"message":"成功"}

打印: response
输出: Response{data={result=数据}, code=1, message='成功'}
```
此时发现序列化和反序列化都没有什么问题，但是我们希望此时的response中的data是Data，而不是泛型了，我们打印下
```java
打印: response.data.getClass()
输出: 报错！无法数据转换
```
那么为什么会报错呢？由于我们定义了response为Data类型，但是gson在反序列化时是不知道的，它只知道我要反序列化一个Response，那么如何解决呢？

Gson内部提供了一个TypeToken
```java
Type type = new TypeToken<Response<Data>>(){}.getType();
Response<Data> response = gson.fromJson(json, type);

打印: type
输出: 包名.Response<包名.Data>
打印: response.data.getClass()
输出: class 包名.Data
```
通过上面打印可以看到通过TypeToken，将Response的T转变为了具体的Data，而TypeToken内部的具体实现就是定义了一个匿名内部类，其中通过发射获取到
TypeToken<Response<Data>>的具体泛型保存到Type中，也就是将Response<Data>保存给Type对象，这个操作就是将T变为了真实的泛型类型，所以Gson
在反序列化的时候可以准确的知道具体是什么泛型类型。

# 实操之通过注解和反射实现findViewById

   初始代码:
   
   ```java
   public class MainActivity extends AppCompatActivity
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

# 实操之通过注解和反射实现getIntent
根据上面的思想，我们来通过注解和反射实现自动getIntent，唯一需要注意的就是实现了Parcelable接口的数组
1. 创建一个注解，用于工具类可以找到
```java 
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface InjectIntent {
    String value() default "";
}
```
2. 在目标Activity中添加注解(如果注解中value没有数值，则直接用名字)
```java
@InjectIntent
String name;

@InjectIntent("attr")
String attr;

@InjectIntent
int[] array;
```
3. 编写工具类
```java
public static void injectIntent(Activity activity) {

        Class<? extends Activity> aClass = activity.getClass();

        //获得数据
        Intent intent = activity.getIntent();
        Bundle extras = intent.getExtras();
        if (extras == null) {
            return;
        }

        Field[] fields = aClass.getDeclaredFields();

        for (Field field : fields) {
            //判断属性是否被打上了我们定义的InjectIntent的注解声明
            if (field.isAnnotationPresent(InjectIntent.class)) {
                //找到使用我们注解的属性，也就是控件
                InjectIntent annotation = field.getAnnotation(InjectIntent.class);

                //获取Intent传递参数的Key
                String key = TextUtils.isEmpty(annotation.value()) ? field.getName() : annotation.value();


                if (extras.containsKey(key)) {

                    Object obj = extras.get(key);

                    //对Parcelable类型的数组特殊处理
                    //获取数组中单个元素类型
                    Class<?> componentType = field.getType().getComponentType();
                    //如果是数组并且是Parcelable的子类
                    if (field.getType().isArray()&& Parcelable.class.isAssignableFrom(componentType)){
                        Object[] objects = (Object[]) obj;
                        //将objects数组通过Arrays拷贝到新的数组中,新的数组就是Parcelable类型
                        //(Class<? extends Object[]>) field.getType() = Parcelable[].class
                        obj = Arrays.copyOf(objects,objects.length,(Class<? extends Object[]>) field.getType());
                    }

                    //通过反射给到控件的值
                    field.setAccessible(true);
                    try {
                        field.set(activity, obj);
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }

                }
            }
        }
    }
```
4. 在目标Activity使用
 ```java
  public class SecActivity extends AppCompatActivity {
    @InjectView(R.id.tv)
    TextView textView;
    
    @InjectIntent
    String name;

    @InjectIntent("attr")
    String attr;

    @InjectIntent
    int[] array;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_sec);
        
        //使用我们的工具自动实现findViewById
        InjectUtils.injectView(this);
        
        //使用我们的工具自动getIntent
        InjectUtils.injectIntent(this);
        
        textView.setText(name);
    }
  }
  ```
  
  # 实操之通过注解反射和动态代理实现onClick
  
  对动态代理不熟悉的可以去[代理模式](../doc/设计模式/代理模式.md)学习

  我们在平时的使用中是
  ```java
  view.setOnClickListener(new View.OnClickListener() {...});
  ```
  如果我们要通过注解，反射和动态代理实现，则要知道几个点
  1. 我们要通过反射的形式自动帮view去调用setOnClickListener(View.OnClickListener())这个方法，同时要代理View.OnClickListener()来执行我们的方法
  2. 由于动态代理只能代理接口，所以我们要添加自己的注解来找到view，以及它调用的setOnClickListener和View.OnClickListener
  3. 拿到方法名setOnClickListener和参数View.OnClickListener，就可以通过view反射找到点击的方法，这时候就可以动态代理

  现在开始码代码:
  1. 添加两个注解
  ```java
  
  //添加第一个元注解
  @Target(ElementType.ANNOTATION_TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface EventType {
    Class listenerType();//用于找到View.OnClickListener().class或者长点击的接口类
    String listenerSetter();//用于找到点击事件的方法名
  }
  
  //添加第二个注解
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  @EventType(listenerType = View.OnClickListener.class, listenerSetter = "setOnClickListener")
  public @interface OnClick {
    int[] value();//该注解可以使用多个id
  }
  
  ```
  
  2. 在Activity中使用注解
  ```java
  @OnClick({R.id.btn1, R.id.btn2})
  public void click(View view) {
        switch (view.getId()) {
            case R.id.btn1:
                Log.i(TAG, "click: 按钮1");
                break;
            case R.id.btn2:
                Log.i(TAG, "click: 按钮2");
                break;
        }
  }
  ```
  
  3. 增加工具类
  ```java
  public class InjectUtils {
    public static void injectEvent(Activity activity) {
        Class<? extends Activity> activityClass = activity.getClass();
        Method[] declaredMethods = activityClass.getDeclaredMethods();
        for (Method method : declaredMethods) {
            //获得方法上所有注解
            Annotation[] annotations = method.getAnnotations();
            for (Annotation annotation : annotations) {
                //注解类型
                Class<? extends Annotation> annotationType = annotation.annotationType();
                //只找符合我们自定义的元注解
                if (annotationType.isAnnotationPresent(EventType.class)) {
                    EventType eventType = annotationType.getAnnotation(EventType.class);
                    // 通过我们的注解元注解拿到注解中的接口，它就是我们要动态代理的接口对象 比如：view.OnClickListener.class
                    Class listenerType = eventType.listenerType();
                    // 通过我们的注解元注解拿到注解中的方法名，它是用于我们自动帮view注册用，比如setOnClickListener
                    String listenerSetter = eventType.listenerSetter();

                    try {
                        // 通过元注解拿到注解方法
                        Method valueMethod = annotationType.getDeclaredMethod("value");
                        // 拿到方法注解的参数，也就是view的id
                        int[] viewIds = (int[]) valueMethod.invoke(annotation);

                        method.setAccessible(true);
                        
                        //创建我们自己的InvocationHandler，用于接收动态代理的回调，等于是我们后面调用view的点击事件，它的参数回调会进入到我们的方法
                        ListenerInvocationHandler<Activity> handler = new ListenerInvocationHandler(activity, method);
                        
                        //创建动态代理，代理view点击事件的回调
                        Object listenerProxy = Proxy.newProxyInstance(listenerType.getClassLoader(),
                                new Class[]{listenerType}, handler);
                        // 遍历注解的值
                        for (int viewId : viewIds) {
                            // 获得当前activity的view（赋值）
                            View view = activity.findViewById(viewId);
                            
                            //通过方法名和参数拿到view中点击事件的方法，因为通过Activity无法找到View内部
                            Method setter = view.getClass().getMethod(listenerSetter, listenerType);
                            // 执行view的该方法，相当于自动帮它去调用注册点击事件，同时回调方法就是我们的动态代理方法
                            setter.invoke(view, listenerProxy); 
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }

        }
    }

    /**
     * 还可能在自定义view注入，所以是泛型： T = Activity/View
     *
     * @param <T>
     */
    static class ListenerInvocationHandler<T> implements InvocationHandler {

        private Method method;
        private T target;

        public ListenerInvocationHandler(T target, Method method) {
            this.target = target;
            this.method = method;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return this.method.invoke(target, args);
        }
    }
  }
  ```
  
  4. 使用:
  ```java
  InjectUtils.injectEvent(this);
  ```

