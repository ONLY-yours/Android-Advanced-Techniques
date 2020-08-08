# MVVM模式

### 1. 什么是MVVM模式

- MVVM是Model-View-ViewModel的简写。它本质上就是MVC 的改进版。MVVM 就是将其中的View 的状态和行为抽象化，让我们将视图 UI 和业务逻辑分开。当然这些事 ViewModel 已经帮我们做了，它可以取出 Model 的数据同时帮忙处理 View 中由于需要展示内容而涉及的业务逻辑。

- 简单点来说，MVVM（Model-View-ViewModel）框架的由来便是MVP（Model-View-Presenter）模式与WPF结合的应用方式时发展演变过来的一种新型架构框架。它立足于原有MVP框架并且把WPF的新特性糅合进去，以应对客户日益复杂的需求变化。

- 微软的WPF：WPF（Windows Presentation Foundation）是微软推出的基于Windows 的用户界面框架，属于.NET Framework 3.0的一部分。它提供了统一的编程模型、语言和框架，真正做到了分离界面设计界面设计人员与开发人员的工作；同时它提供了全新的多媒体交互用户图形界面。

### 2. MVVM模式解析

- Model层：数据服务层，跟其他类MVC模式一样，不管最终对接的是数据库还是网络API或是其它，都是在负责数据的存储，并提供访问数据的接口，以支持数据的增删改查的基本操作。
- View层：界面层，但大家注意，这里的View层相对于MVP模式中的View来说，指代的范围更狭小，原则上它不包含任何界面逻辑，在基本组件库满足需求的情况下，View层的设计和制作完全不需要程序员的参与，所有工作都由界面设计师完成，这也是MVVM的一个核心的思想。对于MVC系列的其他模式，由于界面设计和逻辑开发可以独立的同时进行，第一个好处是开发周期缩短，程序员再不需要等UI将设计图拿给你才开始写上层的功能代码；第二个好处是界面设计师和程序员都可以在更专注的干好自己份内的事。
-  ViewModel层：ViewModel翻译过来—视图的模型，很恰当。ViewModel就是完全反应View的状态和行为，是View的内在抽象。John Gossman 在他的博文中说什么？他说ViewModel包含ViewState、ValueConverter、Commands、DataBindings，所以说
- ViewModel是一个抽象的View一点都没错。我在网上看到很多朋友错误的理解，大家切记ViewModel不是数据模型的封装，不是数据模型的封装，不是数据模型的封装，重说三！从ViewModel的外在属性来看，ViewModel和Model层的数据模型半毛钱关系都没有，它不过是使用了数据模型所携带的数据而已。另外，在MVVM模式的开发设计中，是重View和ViewModel，而轻Model的。当然说轻Model，不是说你Model层就可以随心所欲的设计，而是强调设计师的中心在界面上，程序员的重心在ViewModel上，最后将这精心设计的两层binding起来，就可以保证咱们项目的高大上～。好了，再解释一下上面提到的几个关键词

```
ViewState  指数据的数据状态和显示状态。数据状态就是在视图生命周期中展示的数据的值及其变化；显示状态就是视图在生命周期中显示成什么样的。

ValueConverter  用于格式化数据的，比如需求是将时间显示为昨天今天明天，但是模型中是时间戳，我们就需要ValueConverter来对时间进行格式化。

Commands  包含了视图的所有业务行为，比如登陆操作，就对应一个登陆的command，它将于登陆按钮的点击事件绑定起来，当点击事件发生，command内封装的登陆业务就会自动触发。

DataBindings  不用解释，肯定是指View和ViewModel的绑定了。一般的View中数据容器如textview，跟ViewModel的ViewState中的数据字段绑定起来；View的展示属性跟ViewModel的ViewState中的展示状态绑定起来；View的事件跟ViewModel中定义的命令绑定起来。
```

![image-20200722193258826](C:\Users\i m yours\AppData\Roaming\Typora\typora-user-images\image-20200722193258826.png)



- 上图中，View和ViewModel之间用的是虚线的箭头相连，这表明View和ViewModel没有直接的引用关系，他们各自对于另一方都是透明的。View和ViewModel是在Controller的控制下通过Binding机制绑定在一起，从而协同工作的。

## 快速上手实现

https://www.jianshu.com/p/883027ed4f94

GitHub源地址：

https://github.com/goldze/MVVMHabit

