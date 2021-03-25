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
        //初始mState的状态，枚举State声明了5种状态，后面会去说，此时应该是INITIALIZED
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
        //定义一个集合用于保存观察者方法的信息
        Map<MethodReference, Lifecycle.Event> handlerToEvent = new HashMap<>();
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
            
            //将该方法，callType保存到MethodReference对象中
            MethodReference methodReference = new MethodReference(callType, method);
            
            //将methodReference，方法参数等信息保存到集合中
            verifyAndPutHandler(handlerToEvent, methodReference, event, klass);
        }
        //将集合包装为CallbackInfo并返回
        CallbackInfo info = new CallbackInfo(handlerToEvent);
        mCallbackMap.put(klass, info);
        mHasLifecycleMethods.put(klass, hasLifecycleMethods);
        return info;
}
```
好了，此时我们可以知道绑定的时候大部分工作就是将观察者的事件方法和方法的信息保存到了一个集合中，那什么时候才会触发呢？

我们知道我们的Activity继承自AppCompatActivity，我们去它的父类ComponentActivity中看它的onCreat方法
```java
//ComponentActivity.onCreat
  @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //...
        ReportFragment.injectIfNeededIn(this);
        //...
}

//ReportFragment.injectIfNeededIn
public static void injectIfNeededIn(Activity activity) {
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
}
```
我们可以看到其实就是创建了一个空白的Fragment来监听生命周期，我们去看下ReportFragment的生命周期
```java
@Override
    public void onStart() {
        super.onStart();
        dispatchStart(mProcessListener);
        //其他生命周期一样，都会调用dispatch
        dispatch(Lifecycle.Event.ON_START);
}

private void dispatch(Lifecycle.Event event) {
        Activity activity = getActivity();
        //...

        //我们的Activity都是继承自ComponentActivity，而ComponentActivity又实现了LifecycleOwner
        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                //此时给到LifecycleRegistry的handleLifecycleEvent中
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
}

//LifecycleRegistry.handleLifecycleEvent
public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
        //根据事件获取下一个状态
        State next = getStateAfter(event);
        
        //移动到下一个状态
        moveToState(next);
}
```
我们此时去看看怎么获取的下一个状态
```
static State getStateAfter(Event event) {
        switch (event) {
            case ON_CREATE:
            case ON_STOP:
                return CREATED;
            case ON_START:
            case ON_PAUSE:
                return STARTED;
            case ON_RESUME:
                return RESUMED;
            case ON_DESTROY:
                return DESTROYED;
            case ON_ANY:
                break;
        }
        throw new IllegalArgumentException("Unexpected event value " + event);
    }
```
我们可以看到如果生命周期是
* ON_CREATE和ON_STOP: 它们下一个状态就是CREATED
* ON_START和ON_PAUSE: 它们下一个状态就是STARTED
* ON_RESUME: 它下一个状态就是RESUMED
* ON_DESTROY: 它下一个状态就是DESTROYED
我们第一次初始化状态的时候说的五种状态就是这个

我们再来看看移动到下一个状态
```
 private void moveToState(State next) {
        if (mState == next) {
            return;
        }
        mState = next;
        //...
        mHandlingEvent = true;
        sync();
        mHandlingEvent = false;
}
```
![image](https://user-images.githubusercontent.com/61224872/112408427-ba750d00-8d52-11eb-8de8-b94638742b01.png)
根据上图可以看到可以分为启动过程和销毁过程，mState的取值是根据生命周期发生变化时而变化,比如Activity执行到了OnCreat，那么通过状态可以知道从初始状态到CREATED，通过下面的同步方法，就可以知道观察者是启动过程，那么它的onCreat就会被调用

此时我们进入sync方法
```java
private void sync() {
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        //...
        
        //如果已经同步，则跳出循环
        while (!isSynced()) {
            mNewEventOccurred = false;
            
            //由于集合中保存着观察者的状态，所以根据观察者的状态与下一个状态进行比较，是往前走还是往后走
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                //满足往前走，则进入该方法
                backwardPass(lifecycleOwner);
            }
            Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
                //满足往后走，则进入该方法
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
}

//往左和往右的逻辑差不多，所以我们只看一看往后走
private void forwardPass(LifecycleOwner lifecycleOwner) {

        //找到所有的观察者
        Iterator<Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
                mObserverMap.iteratorWithAdditions();
        
        while (ascendingIterator.hasNext() && !mNewEventOccurred) {
        
            //由于在绑定的时候，已经将观察者的状态进行绑定，所以此时取出
            Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                pushParentState(observer.mState);
                
                //进行事件分发
                observer.dispatchEvent(lifecycleOwner, upEvent(observer.mState));
                popParentState();
            }
        }
}

//ObserverWithState.dispatchEvent
void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = getStateAfter(event);
            mState = min(mState, newState);
            
            //之前绑定的时候，mLifecycleObserver就已经通过反射保存着观察者对应生命周期的方法
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
}

//ReflectiveGenericLifecycleObserver.onStateChanged
@Override
public void onStateChanged(LifecycleOwner source, Event event) {
        //ReflectiveGenericLifecycleObserver中的CallbackInfo中存着方法信息，直接通过反射进行回调
        mInfo.invokeCallbacks(source, event, mWrapped);
}
```



