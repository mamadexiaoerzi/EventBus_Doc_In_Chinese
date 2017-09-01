# EventBus3.0官方文档翻译

> 本文档根据2017/09/01为止最新的官方文档翻译。
>
> 对于拿捏不准的地方，将会贴出原文。有些地方会以qoute的方式注明我本人的理解。如有不正确的地方请大家指出。
>
> [源地址](http://greenrobot.org/eventbus/)

## 简介

EventBus是一个基于发布/订阅模式的，具有低耦合度的Android事件开源库。它使用中央通信机制，使得我们可以只用几行代码就将类与类之间解耦。简化代码，移除依赖，加快开发速度。下图是它的工作原理：

![](https://raw.githubusercontent.com/greenrobot/EventBus/master/EventBus-Publish-Subscribe.png)

## EventBus的优势

- 简化组件之间的通信
- 将事件发送者与事件接收者解耦
- 很好地和Activity，Fragment，后台线程结合
- 避免了复杂的依赖和生命周期问题
- 非常快，为高性能进行了优化
- 非常小（<50k）
- 包含高级特性，如跨线程，订阅者优先级等

## 高级特性

- 基于注解的API

  简单地使用`@Subscribe`注解就可以让订阅者完成事件的订阅。由于采用了编译注解的方式，EventBus不需要在程序运行时进行任何注解反射。

- Android主线程通信

  当需要和UI进行交互时，不管事件是从哪个线程发送出去的，EventBus都可以将事件发送到主线程。

- 后台线程通信

  当你的事件订阅者需要进行耗时操作时，EventBus可以将事件发送到后台线程，以避免UI线程的阻塞。

- 事件和订阅者支持继承

  在EventBus中，事件和订阅者采用面向对象的范式。例如事件B是事件A的子类，那么当一个事件B被发出来时，订阅了事件A的订阅者也会收到该事件。Similarly the inheritance of subscriber classes are considered.

- 快速开始

  不需要额外的设置，你在任何地方使用EventBus.getDefault()就可以获得一个EventBus对象来进行事件操作。

- 支持定制

  通过builder模式，可以对EventBus进行设置以满足你的特殊需求。

  [全部特性](http://greenrobot.org/eventbus/features/)

## 添加依赖

Via Gradle:

```gradle
compile 'org.greenrobot:eventbus:3.0.0'
```

Via Maven:

```xml
<dependency>
    <groupId>org.greenrobot</groupId>
    <artifactId>eventbus</artifactId>
    <version>3.0.0</version>
</dependency>
```

# EventBus文档

- 快速上手
- 跨线程通信
- 设置
- 粘性事件
- 优先级和事件截断
- 。。。

## 快速上手

### 第一步：定义事件

Event类就是简单的Java类，没有任何特殊要求。如：

```java
public class MessageEvent {
    public final String message;
    public MessageEvent(String message) {
        this.message = message;
    }
}
```
### 第二步：订阅事件

在Subscriber中添加处理事件的方法，用以处理收到的事件。这些方法通过添加`@Subscribe`注解来与普通方法区分。

从3.0版本开始，方法名可以随意取，而没有2.0版本中的种种限制。

```java
// This method will be called when a MessageEvent is posted (in the UI thread for Toast)
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessageEvent(MessageEvent event) {
    Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
}
 
// This method will be called when a SomeOtherEvent is posted
@Subscribe
public void handleSomethingElse(SomeOtherEvent event) {
    doSomethingWith(event);
}
```

除了定义事件处理的方法，订阅者还需要向总线进行注册与取消注册。只有注册的订阅者才会收到事件。在Android中，伴随着Activity与Fragment的生命周期的行进，应该在合适的地方进行注册与反注册。通常来说在onStart()和onStop()中进行是个不错的选择。

```java
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}
 
@Override
public void onStop() {
    EventBus.getDefault().unregister(this);
    super.onStop();
}
```

### 第三步：发送事件

在你的代码的任何一个地方发送事件，所有当前处于注册状态，且匹配事件类型的订阅者都会收到该事件。

```java
EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
```

> 译者注：『事件』本质上就是一个包含订阅者所需要信息的对象。由于EventBus拿到事件后是根据订阅者订阅的事件类型来决定将该事件分发给谁，所以订阅者最好能保证自己订阅的事件类型不和别的订阅者雷同，或者将不需要再接收事件的订阅者进行反注册，这样即使存在事件类型雷同的订阅者，只要同时在注册状态的只有一个，也能保证事件的精准送达。
>
> 注意，List<ObjectA>与List<ObjectB>是同一个类，二者只是泛型不同。