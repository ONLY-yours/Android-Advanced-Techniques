# LiveData

## 1. 定义

简单地说，LiveData是一个数据持有类。它具有以下特点：

- 数据可以被观察者订阅；
- 能够感知组件（Fragment、Activity、Service）的生命周期；
- 只有在组件出于激活状态（STARTED、RESUMED）才会通知观察者有数据更新；

PS: 文中提到的“组件”皆指实现了LifecycleOwner接口Fragment、Activity（这个接口用来监听生命周期）。

17年底谷歌推出了正式版，LiveData和ViewModel都是Android官方退出的架构组件（Android Architecture Components）之一。

## 2. LiveData的特性（优点）

- 能够保证数据和UI统一
   这个和LiveData采用了观察者模式有关，LiveData是被观察者，当数据有变化时会通知观察者（UI）。

- 减少内存泄漏
   这是因为LiveData能够感知到组件的生命周期，当组件处于DESTROYED状态时，观察者对象会被清除掉。

- 当Activity停止时不会引起崩溃
   这是因为组件处于非激活状态时，不会收到LiveData中数据变化的通知。

- 不需要额外的手动处理来响应生命周期的变化
   这一点同样是因为LiveData能够感知组件的生命周期，所以就完全不需要在代码中告诉LiveData组件的生命周期状态。

- 组件和数据相关的内容能实时更新
   组件在前台的时候能够实时收到数据改变的通知，这是可以理解的。当组件从后台到前台来时，LiveData能够将最新的数据通知组件，这两点就保证了组件中和数据相关的内容能够实时更新。

- 针对configuration change时，不需要额外的处理来保存数据
    我们知道，当你把数据存储在组件中时，当configuration change（比如语言、屏幕方向变化）时，组件会被recreate，然而系统并不能保证你的数据能够被恢复的。当我们采用LiveData保存数据时，因为数据和组件分离了。当组件被recreate，数据还是存在LiveData中，并不会被销毁。

- 资源共享
   通过继承LiveData类，然后将该类定义成单例模式，在该类封装监听一些系统属性变化，然后通知LiveData的观察者，这个在继承LiveData中会看到具体的例子。

LiveData与ViewModel在MVVM模式中使用，是非常好的选择。

## 3. LiveData的使用

- 使用LiveData对象
- 使用LiveData类

#### 使用LiveData类

LiveData有一个优点，资源共享，直接上代码。

```java
public class MyLiveData extends LiveData<Integer> {
    private static final String TAG = "MyLiveData";
    private static MyLiveData sData;
    private WeakReference<Context> mContextWeakReference;

    public static MyLiveData getInstance(Context context){
        if (sData == null){
            sData = new MyLiveData(context);
        }
        return sData;
    }

    private MyLiveData(Context context){
        mContextWeakReference = new WeakReference<>(context);
    }

    @Override
    protected void onActive() {
        super.onActive();
        registerReceiver();
    }

    @Override
    protected void onInactive() {
        super.onInactive();
        unregisterReceiver();
    }

    private void registerReceiver() {
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(WifiManager.RSSI_CHANGED_ACTION);
        mContextWeakReference.get().registerReceiver(mReceiver, intentFilter);
    }

    private void unregisterReceiver() {
        mContextWeakReference.get().unregisterReceiver(mReceiver);
    }


    private BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            Log.d(TAG, "action = " + action);
            if (WifiManager.RSSI_CHANGED_ACTION.equals(action)) {
                int wifiRssi = intent.getIntExtra(WifiManager.EXTRA_NEW_RSSI, -200);
                int wifiLevel = WifiManager.calculateSignalLevel(
                        wifiRssi, 4);
                sData.setValue(wifiLevel);
            }
        }
    };
}
```

MyLiveData是个继承了LiveData的单例类，在onActive()和onInactive()方法中分别注册和反注册Wifi信号强度的广播。然后在广播接收器中更新MyLiveData对象。在使用的时候就可以通过MyLiveData.getInstance()方法，然后通过调用observe()方法来添加观察者对象，订阅Wifi信息强度变化。

- onActive()，此方法是当处于激活状态的observer个数从0到1时，该方法会被调用。

- onInactive() ，此方法是当处于激活状态的observer个数从1变为0时，该方法会被调用。

简单点来说就是通过Observe观察者模式观察MyLiveData类中的

```java
 private static MyLiveData sData;
```

在网络wifi强度发生变化的时候，通过MyLiveData类中的BroadcastReceiver去接收到信息，然后通过：

```java
sData.setValue(wifiLevel);
```

来通知到Observe的观察者方法：

```java
 MyLiveData.getInstance(this).observe(this, new Observer<Integer>() {
            @Override
            public void onChanged(Integer integer) {
                //通知到observe这个方法
            }
        });
```

## 4. 类关系图

LiveData的类关系图相对比较简单，从上面的类图我们就能看到。和LiveData组件相关的类和接口有：LiveData类、Observer接口、GenericLifecycleObserver接口。LiveData类是个抽象类，但是它没有抽象方法，抽象类有个特点是：不能在抽象类中实例化自己。为什么LiveData会被定义成abstract而又没有抽象方法呢，这个…我也不知道，看了下LiveData的提交记录，是在将hasObservers()替换getObserverCount()方法时将LiveData改成了abstract，在此之前它是被定义为public，可以翻墙的可以看下这里的修改记录

- MediatorLiveData继承自MutableLiveData，MutableLiveData继承自LiveData。MediatorLiveData可以看成是多个LiveData的代理，当将多个LiveData添加到MediatorLiveData，任何一个LiveData数据发生变化时，MediatorLiveData都会收到通知。
- LiveData有个内部类LifecycleBoundObserver，它实现了GenericLifecycleObserver，而GenericLifecycleObserver继承了LifecycleObserver接口。在这里可以回顾下Lifecycle组件相关的内容。当组件（Fragment、Activity）生命周期变化时会通过onStateChanged()方法回调过来。
- Observer接口就是观察者，其中定义了LiveData数据变化的回调方法onChanged()。

## 5. 时序图

LiveData主要涉及到的时序有三个：

- 在Fragment/Activity中通过LiveData.observer()添加观察者（observer()方法中的第二个参数）。
- 根据Fragment/Activity生命周期发生变化时，移除观察者或者通知观察者更新数据。
- 当调用LiveData的setValue()、postValue()方法后，通知观察者更新数据。

## 6. 实现

1. 添加依赖

```xml
// ViewModel and LiveData
    implementation "android.arch.lifecycle:extensions:1.1.0"
// alternatively, just ViewModel
    implementation "android.arch.lifecycle:viewmodel:1.1.0"
// alternatively, just LiveData
    implementation "android.arch.lifecycle:livedata:1.1.0"
```

2. 编写viewmodel

```java
public class MyViewModel extends ViewModel {
        
    //编写需要的逻辑
    //一般viewmodel通过MVVM和界面的数据进行双向绑定

    public MutableLiveData<String> TestObserve = new MutableLiveData<>();

    //编写自己需要的东西，这里之举个简单例子
    public void setTest(String a){
        TestObserve.setValue(a);
    }
    
}
```

3. 在activity中注册并通过observe进行监听

```java
//注册这个ViewModel
MyViewModel myViewModel = ViewModelProviders.of(this).get(MyViewModel.class);

//对ViewModel中的TestObserve进行监听，如果一旦调用了setValue方法，该方法会运行
myViewModel.TestObserve.observe(this, new Observer<String>() {
    @Override
    public void onChanged(String s) {
        //通过setValue方法后调用
        
    }
});
```

逻辑如下：

- 某个位置调用了MyViewModel中的setTest方法：

```java
setTest("test");
```

- 然后在activity中的observe监听事件触发

```java
myViewModel.TestObserve.observe(this, new Observer<String>() {
    @Override
    public void onChanged(String s) {
        //通过setValue方法后调用
        
    }
});
```

## 7.总结

LiveData这个当时退出来其实适合DataBinding一起为了MVVM退出来的，通过DataBinding将页面也ViewMode达到双向绑定的目的。配合Android Google推出的MVVM模式一起使用，开发起来能够完全达成MVVM的目的，提高解耦且使视图控制器完全分离。