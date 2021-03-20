- [1.MVVM](#MVVM)
- [2.MVVM相比MVP](#MVVM相比MVP)

# MVVM

我们如果知道[MVP](./MVP.md)的问题，那么我们就要对[MVP](./MVP.md)中的demo进行近一步优化(使用[DataBinding](.doc/Android/Jetpack/DataBinding/DataBinding.md)，没有接触过得请去[DataBinding](.doc/Android/Jetpack/DataBinding/DataBinding.md)了解
```java
//viewmodel包下MyViewModel.class
public class MyViewModel {
    public ObservableField<String> xx = new ObservableField();

    private Calculate calculate;

    public MyViewModel(){
        calculate = new Calculate();
    }


    public void getData(int x) {
        int i = calculate.add(x);
        xx.set(String.valueOf(i));
    }
}

//view包下Activity
public class MainActivity extends AppCompatActivity {

    private  MyViewModel viewModel = new MyViewModel();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityMainBinding binding = DataBindingUtil.setContentView(this,R.layout.activity_main);
        binding.setViewModel(viewModel);

        viewModel.getData(100);
    }

}
```
