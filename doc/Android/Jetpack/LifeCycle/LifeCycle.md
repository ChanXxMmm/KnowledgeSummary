- [1.使用](#使用)
- [2.解析](#解析)

# 使用
1. 被观察者实现LifeCycleOwner
```java
public class MainActivity extends AppCompatActivity {}
```

2. 观察者实现LifeCycleObserver，通过注解的方式实现观察生命周期的方法,方法名随意
```java
public class MyObserver implements LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public void onCreat(){
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public void onStart(){
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void onResume(LifecycleOwner owner){
    }
    
    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void onPause(){
    }
    
    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void onStop(){
    } 
    
    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    public void onDestory(){
    }
    
    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    public void OnAny(LifecycleOwner owner,Lifecycle.Event event){
    }
}
```
3. 被观察者调用getLifeCycle().addObserver(观察者)
```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        getLifecycle().addObserver(new MyObserver());
    }
}
```

# 解析
我们知道它是用注解，反射和[观察者模式](../../../设计模式/观察者模式.md)实现的，下面我们看下它到底是如何实现的

我们从绑定入口开始分析:
```java
入口:  getLifecycle().addObserver(new MyObserver());

//进入addObserver方法，会发现是抽象类Lifecycle的方法，我们进入实现类LifecycleRegistry中查看
//LifecycleRegistry.addObserver
@Override
    public void addObserver(@NonNull LifecycleObserver observer) {
        //初始mState的状态，枚举State声明了5中状态，后面会去说，此时应该是INITIALIZED
        State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
        
        //将观察者observer和初始状态包装为ObserverWithState
        ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
        
        //......
    }
```
此时我们进入ObserverWithState会发现他是一个静态内部类
```java
static class ObserverWithState {
        State mState;
        LifecycleEventObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
            //将观察者observer通过Lifecycling.lifecycleEventObserver返回一个mLifecycleObserver对象
            mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
            //......
        }
        
        //......
}
```
我们来看看Lifecycling.lifecycleEventObserver到底干了什么呢
```java
//Lifecycling.lifecycleEventObserver
@NonNull
    static LifecycleEventObserver lifecycleEventObserver(Object object) {
        //......
        
        return new ReflectiveGenericLifecycleObserver(object);
}

//ReflectiveGenericLifecycleObserver的构造函数
ReflectiveGenericLifecycleObserver(Object wrapped) {
        mWrapped = wrapped;
        //将观察者的字节码给到ClassesInfoCache处理
        mInfo = ClassesInfoCache.sInstance.getInfo(mWrapped.getClass());
}

//ClassesInfoCache.sInstance.getInfo
CallbackInfo getInfo(Class<?> klass) {
        //从Map中获取缓存观察者字节码的包装类信息CallbackInfo
        CallbackInfo existing = mCallbackMap.get(klass);
        
        //首次肯定是没有的
        if (existing != null) {
            return existing;
        }
        
        //进入createInfo通过观察者字节码来包装成为CallbackInfo
        existing = createInfo(klass, null);
        return existing;
}

//ClassesInfoCache.sInstance.createInfo
private CallbackInfo createInfo(Class<?> klass, @Nullable Method[] declaredMethods) {
        //......

        //通过反射获取观察者中所有声明的方法
        Method[] methods = declaredMethods != null ? declaredMethods : getDeclaredMethods(klass);
        boolean hasLifecycleMethods = false;
        for (Method method : methods) {
        
            //得到使用OnLifecycleEvent的注解
            OnLifecycleEvent annotation = method.getAnnotation(OnLifecycleEvent.class);
            
            //......
            
            //获得方法的参数
            Class<?>[] params = method.getParameterTypes();
            int callType = CALL_TYPE_NO_ARG;
            
            //因为我们方法的参数是1个，所以肯定callType = CALL_TYPE_PROVIDER;
            if (params.length > 0) {
                callType = CALL_TYPE_PROVIDER;
                //......
            }
            
            //获得OnLifecycleEvent注解的参数
            Lifecycle.Event event = annotation.value();

            //......
            
            //将该方法与callType保存到MethodReference对象中
            
            
            MethodReference methodReference = new MethodReference(callType, method);
            verifyAndPutHandler(handlerToEvent, methodReference, event, klass);
        }
        CallbackInfo info = new CallbackInfo(handlerToEvent);
        mCallbackMap.put(klass, info);
        mHasLifecycleMethods.put(klass, hasLifecycleMethods);
        return info;
}
 
```
