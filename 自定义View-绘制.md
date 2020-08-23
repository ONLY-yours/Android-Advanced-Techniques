# 自定义View-绘制

自定义view中大致有三个板块：

- 布局
- 绘制
- 触摸反馈

### 1. 绘制

指的是控件绘制，一般来说我们通过API来操作控件在界面上显示。

绘制也是自定义View的灵魂。

如果想查看数据，类似于数据表，使用textview就能简单的绘制出来，但是如果想将数据以直方图、饼图、折线图等来表达出来，通过控件和布局拼接就会显得很复杂，然后自定义绘制就能帮助我们简单制作出来。

##### 自定义绘制的实现

Android的绘制都是在控件的绘制方法中。

```java
@RemoteView
public class TextView extends View implements OnPreDrawListener {

	//绘制方法
   protected void onDraw(Canvas canvas) {
        throw new RuntimeException("Stub!");
    }


}
```

绘制方法并不是一个方法，而是有好几个，其中最常用的就是onDraw()方法。

这个onDraw()方法，负责的是View的主体绘制。例如TextView的文字、ImageView的图像，都是在onDraw()里面绘制的。

具体执行绘制的是onDraw()里面的Canvas参数。

Android界面基本上都是由Canvas绘制而成的，除了一些游戏等是通过OpenGL绘制。

```java
Canvas.class:

public void drawPicture(Picture picture) {
        throw new RuntimeException("Stub!");
    }

public void drawText(String text, float x, float y, Paint paint) {
        throw new RuntimeException("Stub!");
    }
    
public void drawBitmap(int[] colors, int offset, int stride, int x, int y, int width, int height, boolean hasAlpha, Paint paint) {
        throw new RuntimeException("Stub!");
    }

```

Canvas通过这些方法来绘制界面。

通过drawCircle来举个例子

```java
 public void drawCircle(float cx, float cy, float radius, Paint paint) {
        throw new RuntimeException("Stub!");
    }
```

cx，cy指的是圆心位置，radius为半径，paint本身为颜料，这个提供颜色和风格，风格指的是圆的状态，实心还是空心，线条粗细，是否有阴影等。

paint是非常重要的组件信息，这个是灵魂，包括其他的draw方法，只要填入自己需要的paint信息就行了

除了draw用来绘制，还有一些方法帮助绘制。

- 绘制范围的裁切，将绘制出来的东西按照自己想要的格式进行裁切，这些方法一般以clip-开头
- 绘制内容的几何变换，放大缩小、旋转、平移、错切、水平错切和垂直错切。

虽然常用的绘制方法是onDraw()，但是有些情况下需要其他的绘制方法，在Android中，后绘制的会将前面绘制的给覆盖掉，如果想要特定位置，比如说位于顶部、背景等位置，就需要去查询到底是哪个绘制函数负责，然后重写绘制函数，编写你要的paint。

#### 总结

- 需要自定义View需要重写绘制方法（onDraw()）
- paint是重要的参数，可以帮助你完成想要的效果
- 通过辅助绘制（裁切和几何变换）完成你想要的效果
- 如果对覆盖关系有特别的要求，选择你想要的绘制方法重写