- [1.MVP](#MVP)
- [2.MVP相比MVC](#MVP相比MVC)
- [3.应用场景](#应用场景)
# MVP

我们如果知道[MVC](./MVC.md)的问题，那么我们就要对[MVC](./MVC.md)中的demo进行近一步优化
```java
//view包下MyView
public interface MyView {
    void setData(int a);
}

//presenter包下MyPresenter
public class MyPresenter {
    private Calculate calculate;
    private MyView view;

    public MyPresenter(MyView v){
        this.view = v;
        calculate = new Calculate();
    }


    public void getData() {
        int i = calculate.add(8);
        view.setData(i);
    }
}

//view包下Activity
public class MainActivity extends AppCompatActivity implements MyView{

    private TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = findViewById(R.id.tv);

        MyPresenter presenter = new MyPresenter(this);
        presenter.getData();
    }

    @Override
    public void setData(int a) {
        textView.setText(String.valueOf(a));
    }
}
```
此时我们可以看到我们将Activity中与model相关的交给了Presenter，Activity只负责事件，而Presenter负责具体的逻辑，这就是MVP

* 数据(Model): 数据+对数据进行的操作(不依赖Activity的操作)
* 视图(View) : xml，MyView接口，Activity
* 逻辑(Controller): MyPresenter

# MVP相比MVC

* 进步: Activity只剩了下View，presenter承担了view和model之间的交互，对视图数据逻辑的分离是清晰地，满足了单一职责原则
* 缺陷: 引入了接口，导致方法增多，每增加一个方法要改几个地方

# 应用场景

核心，复杂，需求变更快页面


