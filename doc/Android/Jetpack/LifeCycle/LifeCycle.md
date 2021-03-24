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
我们知道它是用注解，反射和[观察者模式](../doc/设计模式/观察者模式.md)实现的，下面我们看下它到底是如何实现的

我们从绑定入口开始分析:
