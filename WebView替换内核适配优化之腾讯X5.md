# WebView替换内核适配优化之腾讯X5

Android原生的webview在对H5的支持并不是很好。而且自从Android 4.4之后，默认的浏览器内核从WebKit转化为chromium，因此适配什么的会遇到很多瓶颈。

目前可选用的内核又两种，一个是腾讯的X5，一个是crosswalk。这次采用X5，相比crosswalk，x5接入速度快且占有体积不大（crosswalk会导致安装包多20m左右）

## 1. 接入

详情请见：

https://x5.tencent.com/docs/access.html

我采用直接导入的方式接入（且我的项目没有代码混淆和异常上报，所有这两步骤掠过）

#### 1.1 SDK接入

- jar包方式集成 您可将官网下载的jar包复制到您的App的libs目录，并且通过Add As Library的方式集成TBS SDK

- Gradle方式集成 您可以在使用SDK的模块的dependencies中添加引用进行集成：

  ```
  api 'com.tencent.tbs.tbssdk:sdk:43903'
  ```

#### 1.2 权限配置

为了保障内核的动态下发和正常使用，您需要在您的AndroidManifest.xml增加如下权限：

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```

#### 1.3 首次初始化冷启动优化

TBS内核首次使用和加载时，ART虚拟机会将Dex文件转为Oat，该过程由系统底层触发且耗时较长，很容易引起anr问题，解决方法是使用TBS的 ”dex2oat优化方案“。

（1）. 设置开启优化方案

```java
// 在调用TBS初始化、创建WebView之前进行如下配置 
HashMap map = new HashMap(); 
map.put(TbsCoreSettings.TBS_SETTINGS_USE_SPEEDY_CLASSLOADER, true); 
map.put(TbsCoreSettings.TBS_SETTINGS_USE_DEXLOADER_SERVICE, true); 
QbSdk.initTbsSettings(map);
```

（2）. 增加Service声明

1. 在AndroidManifest.xml中增加内核首次加载时优化Service声明。

2. 该Service仅在TBS内核首次Dex加载时触发并执行dex2oat任务，任务完成后自动结束。

   ```xml
   <service 
   android:name="com.tencent.smtt.export.external.DexClassLoaderProviderService"
   android:label="dexopt"
   android:process=":dexopt" >
   </service>
   ```

####  1.4 配置网络

```xml
//允许http明文传输
android:networkSecurityConfig="@xml/network_security_config"

<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```

## 2. 编写界面

```xml
<com.tencent.smtt.sdk.WebView

    android:id="@+id/full_web_webview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"

    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    ></com.tencent.smtt.sdk.WebView>
```

## 3. 编写Activity

```java
@Override
protected void onCreate(Bundle savedInstanceState) {

    //冷启动优化
    HashMap map = new HashMap();
    map.put(TbsCoreSettings.TBS_SETTINGS_USE_SPEEDY_CLASSLOADER, true);
    map.put(TbsCoreSettings.TBS_SETTINGS_USE_DEXLOADER_SERVICE, true);
    QbSdk.initTbsSettings(map);

    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    txwebView = findViewById(R.id.full_web_webview);
    txwebView.onResume();

    initsetting();

    //loadUrl中填写需要嵌入的网址
    txwebView.loadUrl("http://www.baidu.com");
  
  	·········
  }
```

在initsetting中对浏览器的setting进行设置（根据自己的要求可增加或减少）：

```java
void initsetting(){
    WebSettings webSetting =txwebView.getSettings();
    webSetting.setJavaScriptEnabled(true);
    webSetting.setJavaScriptCanOpenWindowsAutomatically(true);
    webSetting.setAllowFileAccess(true);
    webSetting.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.NARROW_COLUMNS);
    webSetting.setSupportZoom(true);
    webSetting.setBuiltInZoomControls(true);
    webSetting.setUseWideViewPort(true);
    webSetting.setSupportMultipleWindows(true);
    webSetting.setAppCacheEnabled(true);
    webSetting.setDomStorageEnabled(true);
    webSetting.setGeolocationEnabled(true);
    webSetting.setAppCacheMaxSize(Long.MAX_VALUE);
    webSetting.setPluginState(WebSettings.PluginState.ON_DEMAND);
    webSetting.setCacheMode(WebSettings.LOAD_NO_CACHE);

    //支持通过js打开新窗口
    webSetting.setJavaScriptCanOpenWindowsAutomatically(true);
    webSetting.setLoadsImagesAutomatically(true); //支持自动加载图片
    webSetting.setDefaultTextEncodingName("utf-8");//设置编码格式
}
```

下面对这个x5webview监听事件和原生的webview没有区别：

```java
txwebView.setWebViewClient(new WebViewClient(){
    
    //只在当前webview中打开界面，不打开第三方界面
    @Override
    public boolean shouldOverrideUrlLoading(WebView webView, String s) {
        webView.loadUrl(s);
        return super.shouldOverrideUrlLoading(webView, s);
    }

    @Override
    public void onPageStarted(WebView webView, String s, Bitmap bitmap) {
        //设定加载开始的操作
        super.onPageStarted(webView, s, bitmap);
    }

    //加载结束后的事件
    @Override
    public void onPageFinished(WebView webView, String s) {
        //设定加载结束的操作
   
        super.onPageFinished(webView, s);
    }

    //网页报错的解决办法
    @Override
    public void onReceivedError(WebView webView, int i, String s, String s1) {
       
        super.onReceivedError(webView, i, s, s1);
    }
});
```