# Android设计模式-观察者模式

### 1.定义

定义对象间的一种一个对多的依赖关系，当一个对象的状态发送改变时，所以依赖于它的对象都得到通知并被自动更新。

### 2.介绍

- 观察者属于行为型模式。
- 观察者模式又被称作发布/订阅模式。
- 观察者模式主要用来解耦，将被观察者和观察者解耦，让他们之间没有没有依赖或者依赖关系很小。

### 3. UML类图

![image-20200730195208573](C:\Users\i m yours\Documents\Andriod开发日志及笔记\Android进阶计划\进阶技术学习\image\1.png)

##### 角色说明：

- Subject（抽象主题）：又叫抽象被观察者，把所有观察者对象的引用保存到一个集合里，每个主题都可以有任何数量的观察者。抽象主题提供一个接口，可以增加和删除观察者对象。
- ConcreteSubject（具体主题）：又叫具体被观察者，将有关状态存入具体观察者对象；在具体主题内部状态改变时，给所有登记过的观察者发出通知。
- Observer (抽象观察者):为所有的具体观察者定义一个接口，在得到主题通知时更新自己。
- ConcrereObserver（具体观察者）：实现抽象观察者定义的更新接口，当得到主题更改通知时更新自身的状态。

### 4.模式实现

观察者模式定义：定义了对象之间的一对多依赖，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。

观察者模式其实有两种实现方式：

1. 使用JDK封装好的观察者模式设计API
2. 不使用JDK封装好的API，自己实现观察者模式

#### 使用JDK封装的API

```java
WatchedOne activityOrNot = new WatchedOne();		//创建一个被观察者实例

WatchObserver observer = new WatchObserver();		//创建一个观察者
	
activityOrNot.addObserver(observer);				//为被观察者加入上面定义的观察者

activityOrNot.Onupdate();							//被观察者改变（Onupdate是写好的方法）会调用观察者的里面方法
```

WatchedOne.class（被观察者类）

```java
//被观察者
public class WatchedOne extends Observable {

    public void Onupdate(){
        //写自己需要的逻辑
    }

}
```

WatchObserver.class（观察者类）

```java
//观察者
public class WatchObserver implements java.util.Observer {

    @Override
    public void update(Observable observable, Object o) {
        System.out.println("更新辣，内容是："+o);
    }

}
```

简单点来说就是当被观察者自己被调用的时候（被观察者中的任意一个方法），观察者都会执行里面的update方法。

### 5.总结

**1、可以一对多，也可以多对一；**
**2、稳定的消息更新传递机制，并抽象了更新接口；**
**3、对象解耦(架构、模式啥的不都是为了解耦嘛，汗)；**
**4、EventBus、广播、RxJava等都是使用观察者模式，它们使用场景要了解，而JDK已经提供了观察者模式的API,使用也不比它们难**

**5、在MVVM设计模式中，使用SingleLiveEvent配合Observer使用会更加高效**