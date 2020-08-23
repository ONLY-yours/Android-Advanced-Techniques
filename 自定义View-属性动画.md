# 自定义View-属性动画

原理：

不断更新View的属性，以此达到动画效果，这就是属性动画的原理。

简答来说，你想让一个ImageView在界面上从做移到右边，直接通过

```java
imageView.setTranslationX(500);
```

这样这个imageview就直接移动了500，但是中间过程是没有的。我们给他一个for循环，每次向右移动10，做50次，这样移动的动画就做好了。（这样的动画不能放在主线程更新，会导致阻塞）这样就是一个属性动画原理。

但是如果复杂的动画效果，比如不是一个匀速运动，是一个减速过程。移动并不是以直线，是以曲线运动等等问题，如果只是简单的通过这样子去设置动画，这样的效果是很难达到的。（当然理论上手写代码都可以达到这样的效果）

但是如果使用属性动画的API，那么这些效果就能很快达成。

#### 属性动画API使用

### ViewPropertyAnimator

这个是属性动画里面最简单的工具，调用view.animator()，让后在加几行代码即可。

我们上面那个移动的逻辑，就可以这样子写：

```java
imageView.animate().translationX(500);
```

然后这个api会自动每隔10ms调用一次setTranslationX方法，一点点挪动位置，直到帮我们移动到右边500像素的位置。

- 原理：我们在调用imageView.animate()时候，本质上调用了一个ViewPropertyAnimator的对象。

![translation](image\translation.jpg)

同时编写三四个方法就能同时改变三四个属性。

```java
    imageView.animate()
                .translationX(500)
                .rotation(90)
                .scaleX(2);
```

同时也可以改变动画的时长，默‘

认动画时常为300ms，可以通过setDuration来指定动画时长，默认ms。

```java
imageView.animate().setDuration(500);
```

当然也可以改变动画变化的模型，简单点来说，假设为匀速改变，比如从0位移到500，10ms移动100距离，如果是匀速的话，移动到300位置的时候，时间过去了30ms，两者是对应的。但是我们可以设置其为不是匀速模型。方法为setInterpolator（）

```java
//方法源代码如下：
public ViewPropertyAnimator setInterpolator(TimeInterpolator interpolator) {
    throw new RuntimeException("Stub!");
}
```

你可以通过指定interpolator来指定对应的模型，减速模型加速模型等等。（这个过程称为内插）

默认的内插模型为AccelerateDecelerateInterpolator，指的是开始阶段加速，在结束阶段减速的模型。

- LinearInterpolator			匀速模型
- AccelerateInterpolator     持续加速
- DecelerateInterpolator    持续减速
- AnticipateInterpolator      蓄力模型（我比较喜欢这么叫，指的是先回拉动画，然后在进行，看起来有点像弹弓蓄力先向后，然后弹出）
- OvershootInterpolator    回弹模型（这个和上面的蓄力模型正好相反，这个是会运动过头，然后最后再回弹回合适的位置）
- AnticipateOvershootInterpolator   蓄力后回弹模型（上面两个的结合）
- BounceInterpolator         弹力球模型（就像弹力球一样从高处落下，在结束的位置会有多次回弹效果，直至停止）

还有各种各样的动画模型，还有自定义动画完成度的模型。（感觉和视频剪辑中的调整时间曲线有些类似的地方）

#### 设置监听器

```java
   imageView.animate()
                .setDuration(500)
                .setInterpolator(new Interpolator())
                .translationX(500)
                .rotation(90)
                .scaleX(2)
				.setListener(new Animator.AnimatorListener() {
                    //动画开始的时候监听
                    @Override
                    public void onAnimationStart(Animator animator) {

                    }
					//动画结束的时候监听
                    @Override
                    public void onAnimationEnd(Animator animator) {

                    }
					//动画取消的时候监听
                    @Override
                    public void onAnimationCancel(Animator animator) {

                    }
					//动画更新的时候监听
                    @Override
                    public void onAnimationRepeat(Animator animator) {

                    }
                });
```

当然还有各种各样的监听，具体请见：

https://hencoder.com/ui-1-6/

### ObjectAnimator

这个相比于上面的animator，这个可以自定义属性，上面的只能用他给的几种api来使用，无论是给好的控件或是自定义View，都不能做复杂的自定义操作。

这个虽然可以自定义属性，但是相比于上面ViewPropertyAnimator，使用起来也更加复杂。

1. 给自定义View添加setter/getter方法

2. 使用ObjectAnimator.ofXXX来创建对象。

3. ObjectAnimator..start()开始动画

同时上面设置时常，速度模型，监听器和上面的ViewPropertyAnimator也是一样的，也有。