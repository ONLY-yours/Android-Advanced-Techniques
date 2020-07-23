# MVVM模式之-入门DataBinding的使用

### 1. 使用步骤

1.1 引入Data Binding函数库

在build .gradle中使用如下设置支持dataBinding:

```groovy
dataBinding{
    enabled = true

}
```

1.2 新建一个bean类

```java
public class TestBean {
    private String time;
    private String location;
    private String phone;

    public String getLocation() {
        return location;
    }

    public String getPhone() {
        return phone;
    }

    public String getTime() {
        return time;
    }

    public void setLocation(String location) {
        this.location = location;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public void setTime(String time) {
        this.time = time;
    }
}
```

2.3 编写xml布局

```xml
<?xml version="1.0" encoding="utf-8"?>

    <layout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    xmlns:binding="http://schemas.android.com/apk/res-auto">

        <data>
            <import type="com.example.databindingprojectofmvvm.TestBean"/>

            <variable
                name="TestModule"
                type="TestBean"
            />
        </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">


     <Button
            android:id="@+id/btn_test"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{controller::onclick}"
            binding:layout_constraintTop_toTopOf="parent"
            binding:layout_constraintLeft_toLeftOf="parent"
            binding:layout_constraintRight_toRightOf="parent"/>

    <TextView
        android:id="@+id/tv_phone"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{TestModule.phone}"
        binding:layout_constraintBottom_toTopOf="@id/tv_time"
        binding:layout_constraintLeft_toLeftOf="parent"
        binding:layout_constraintRight_toRightOf="parent"
        binding:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/tv_time"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{TestModule.time}"
        binding:layout_constraintBottom_toTopOf="@id/tv_location"
        binding:layout_constraintLeft_toLeftOf="parent"
        binding:layout_constraintRight_toRightOf="parent"
        binding:layout_constraintTop_toBottomOf="@id/tv_phone"/>

    <TextView
        android:id="@+id/tv_location"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{TestModule.location}"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/tv_time" />

    
    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

布局中我们要注意的是这个data便签，其中type字段是数据绑定对应的实体类，name就是我们引用的一个属性标志，在这里写为TestModul，如果要给tv_location赋值，我们直接通过@{TestModul.属性},相当于把TestBean属性变量赋值给id为tv_location的TextView。

2.4 在Activity中引用

以为我们已经配置了databinding属性为true，所以会自动为我们生成Binding类，生成规则为布局名后面加Binding，比如我们这里的布局名称是activity_main，生成的Bingding类就是ActivityMainBinding，我们在Activity中通过下列代码使用：

```java
 private TestBean testBean = new TestBean();


@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initview();

    ActivityMainBinding binding = DataBindingUtil.setContentView(this,R.layout.activity_main);
    
    //该点击事件无效
    findViewById(R.id.btn_test).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            testBean.setLocation("location:hangzhou");
            testBean.setPhone("18329036023");
            testBean.setTime("2020-7-23");
        }
    });
    testBean.setTime("before");
    binding.setTestModule(testBean);
}
```

这个时候我们发现tv_time上已经设置了before字段，然后我编写了一串点击事件，点击后发现是无效的。经过测试后，发现是onclick的时候确实将testBean设置了，只不过绑定后并没有更新UI。只需要将上面的点击事件加入绑定即可。

```java
findViewById(R.id.btn_test).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {

        testBean.setLocation("location:hangzhou");
        testBean.setPhone("18329036023");
        testBean.setTime("2020-7-23");

        binding.setTestModule(testBean);
    }
});
```