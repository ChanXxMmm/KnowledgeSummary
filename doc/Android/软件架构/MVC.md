- [1.如何拆分代码](#如何拆分代码)
- [2.MVC](#MVC)
- [3.MVC对比一件打天下](#MVC对比一件打天下)

# 如何拆分代码
相比于一把梭写代码，我们要分清什么是数据，什么是视图，以及什么是逻辑
* 数据(Model): 数据+对数据进行的操作(不依赖视图的操作)
* 视图(View) : 不同的模式有不同的定义(xml+Activity+Fragment = View合集)
* 逻辑(Controller): view和model的通信和交互

# MVC
我们先来看下一把梭的简单代码
```java
public class MainActivity extends AppCompatActivity {

    private TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = findViewById(R.id.tv);

        getData();
    }

    private void getData() {
        int a = 0;
       //.....一顿复杂计算
        
        setData(String.valueOf(a));
    }

    private void setData(String data){
        textView.setText(data);
    }
}
```
上面的代码很简单，就是一顿复杂计算将结果展示在TextView上，我们对他进行结构优化到MVC
```java
//model/Calculate.java
public class Calculate {

    public int add(int i){
        //一顿操作
        return 9;
    }
}

//MainActivity.java
public class MainActivity extends AppCompatActivity {

    private TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = findViewById(R.id.tv);

        getData();
    }

    private void getData() {
        Calculate calculate = new Calculate();
        int i = calculate.add(8);
        textView.setText(String.valueOf(i));
    }
}

```
此时我们可以看到我们将与Activity无关的操作抽离到model包下，这就是MVC
* 数据(Model): 数据+对数据进行的操作(不依赖Activity的操作)
* 视图(View) : xml
* 逻辑(Controller): Activity(负责view的初始化，负责model与view的沟通)

# MVC对比一件打天下
MVC相比于一把梭:
* 进步: 抽离了Model,对数据进行了剥离
* 缺陷: Controller的权利太大(Activity)，因为Activity负责了view和model的创建，导致了什么事情都在Activity做，当需求改变时Activity会变得很臃肿，又变成了一件打天打




