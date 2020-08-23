# 自定义View-布局

一般情况下不需要在意这个布局，所有的自带控件的布局过程算法都已经写好了。只需要编写xml和布局文件，当运行时，会自动对这些View计算出尺寸和位置。这个过程完全自动。

### Android布局过程

- 测量过程：onMeasure（）
- 布局过程：onLayout（）

先计算，从上到下，从根部到顶部，逐步计算其中的位置，然后将View布局上去。

#### 具体：

1. 重写onMeasure（）来修改已有View 的尺寸
2. 重写onMeasure（）来全新计算自定义View的尺寸
3. 重写onMeasure（）和onLayout（）来全新计算自定义ViewGroup的内部布局

### 第一类自定义的具体做法

也就是重写 `onMeasure()` 来修改已有的 `View` 的尺寸的具体做法：

1. 重写 `onMeasure()` 方法，并在里面调用 `super.onMeasure()`，触发原有的自我测量；
2. 在 `super.onMeasure()` 的下面用 `getMeasuredWidth()` 和 `getMeasuredHeight()` 来获取到之前的测量结果，并使用自己的算法，根据测量结果计算出新的结果；
3. 调用 `setMeasuredDimension()` 来保存新的结果。