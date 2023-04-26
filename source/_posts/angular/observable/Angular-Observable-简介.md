---
title: Angular Observable 简介
date: 2023-04-14 11:51:36
tags:
---
# 数据流

流是在一段时间内到达的数据。数据流可以是任何东西，比如变量、用户输入、属性、缓存、数据结构，甚至异常等。

考虑一下鼠标单击事件的x和y位置序列的例子。假设用户已经按顺序点击了位置（12,15）、（10,12）、（15,20）和（17,15）。下图显示了值是如何在一段时间内到达的。正如您所看到的，流在值发生时即异步地发出。

值并不是流发出的唯一东西，流可以在用户关闭窗口或应用程序时完成，或者程序发生错误时，导致流关闭。任何时间，流都可能发出以下三种东西中的任何一种。

+ Value：即流中的下一个值
+ Complete：流已结束
+ Error：错误已停止流

正如前面所说，数据流可以是任何东西。例如

+ 鼠标点击或鼠标悬停事件
+ 键盘事件，如键被按下、键被抬起、按键等
+ 表单事件，如值被跟新等
+ HTTP请求后到达的数据
+ 用户通知

现在，我们已经了解了什么是数据流，让我们看看什么是响应式编程

# 响应式编程
响应式编程就是创建数据流，触发值，错误或完成信号。修改、转换或对数据流做一些有用的操作。

# 什么是RxJS
RxJS（Reactive Extensions Library for JavaScript）是一个JavaScript库，它允许我们使用异步数据流

Angular在其框架中大量使用RxJS库来实现响应式编程。使用响应式编程的一些示例如下：

+ Angular中响应HTTP请求
+ Angular中表单中值/状态变更
+ Router和Forms模块使用可观察性来监听和响应用户输入事件
+ 通过自定义事件从子组件发送observable输出数据到父组件
+ HTTP模块使用Observable来处理AJAX请求和响应

RxJs有两个主要对象
+ 可观察者(Observable)
+ 观察者（Observers/Subscribers)

# 什么是Angular中的可观察者(Observable)
Observable是一个将普通数据流转换为可观测数据流的函数。您可以将Observable看作是普通数据流的包装器。

Observable流或简单Observable使用异步流发送值内容，当流发送完成时，它会发出完成信号，当流出错时，他会发出错误信号。

Observable是声明性的，定义了一个Observable函数，就像定义其他变量一样。只有当有人订阅了observable时，它才会开始发出值。

# 谁是观察者(subscribers)

“Observable”本身是无用的，除非有人消耗了Observable发出的值。我们称他们为观察者或订阅者。观察者使用回调与Observable通信。

观察者必须订阅可观察的事物，才能从可观察的东西中获得值。在订阅时，它可以选择性地传递这三个回调。next（）、error（）和complete（）

一旦观察者或消费者订阅了该值，可观察对象就开始发出该值。

每当值到达流中时，observable就会调用next()回调。它将该值作为参数传递给next()回调。如果发生错误，则调用error()回调。当流完成时，它调用complete()回调。

+ 观察者/订阅者订阅Observables
+ 观察者/订阅者在订阅时向observable注册三个回调。next()、error()和complete()
+ 所有三个回调都是可选的
+ 观察器通过next()回调从观察器接收数据
+ 他们还通过error()和complete()回调从Observable接收错误和完成事件

# 创建 Observable 对象

有几种方法可以在angular中创建可观察的对象。最简单的是使用Observable构造函数。observable构造函数以subscriber作为其参数。subscriber将在这个observable的subscribe()方法执行时运行。

以下示例创建了一个数字流1、2、3、4、5的observable

~~~ts
obs = new Observable((observer) => {
     console.log("Observable starts")
     observer.next("1")
     observer.next("2")
     observer.next("3")
     observer.next("4")
     observer.next("5")
   })
~~~

变量obs现在属于observable类型。

上面的例子声明obs为observable，但没有实例化它。要使observable发出值，我们需要订阅它。

在上面的例子中，我们使用了Observable构造函数来创建Observable。RxJS库有许多可用的运算符，这使得创建可观察对象变得容易。这些运算符帮助我们从数组、字符串、promise、任何可迭代的内容等,创建可观察的内容。

# 订阅可观测对象

我们通过调用observable上的subscribe方法来订阅它。我们可以选择性地包括三个回调next()、error()和complete()，如下所示
~~~ts
ngOnInit() {
 
    this.obs.subscribe(
      val => { console.log(val) }, //next callback
      error => { console.log("error") }, //error callback
      () => { console.log("Completed") } //complete callback
    )
}
~~~

# 添加间隔

~~~ts
obs = new Observable((observer) => {
    console.log("Observable starts")
 
    setTimeout(() => { observer.next("1") }, 1000);
    setTimeout(() => { observer.next("2") }, 2000);
    setTimeout(() => { observer.next("3") }, 3000);
    setTimeout(() => { observer.next("4") }, 4000);
    setTimeout(() => { observer.next("5") }, 5000);
    
  })
~~~

# Error 事件

如前所述，Observable也会发出误差。这是通过调用error（）回调并传递error对象来完成的。Observable在发出错误信号后停止。因此，永远不会发射值4和5。

~~~ts
obs = new Observable((observer) => {
    console.log("Observable starts")
 
    setTimeout(() => { observer.next("1") }, 1000);
    setTimeout(() => { observer.next("2") }, 2000);
    setTimeout(() => { observer.next("3") }, 3000);
    setTimeout(() => { observer.error("error emitted") }, 3500);    //sending error event. observable stops here
    setTimeout(() => { observer.next("4") }, 4000);          //this code is never called
    setTimeout(() => { observer.next("5") }, 5000);
    
  })
 ~~~

 # 完成事件

 在发出完成信号后，Observable就会停止，因此，永远不会发射值4和5。
~~~ts
    obs = new Observable((observer) => {
    console.log("Observable starts")
 
    setTimeout(() => { observer.next("1") }, 1000);
    setTimeout(() => { observer.next("2") }, 2000);
    setTimeout(() => { observer.next("3") }, 3000);
    setTimeout(() => { observer.complete() }, 3500);   //sending complete event. observable stops here
    setTimeout(() => { observer.next("4") }, 4000);    //this code is never called
    setTimeout(() => { observer.next("5") }, 5000);
    
  })
  ~~~

  # Observable 操作符

操作符是对Observable进行操作并返回新Observable的函数。

Observable的能力来自于操作符，您可以使用操作符来操作传入的Observable对象，对其进行过滤，将其与另一个Observable对象合并，更改值或订阅另一个Observable对象。

也可以使用管道将操作符链接起来。链中的每个操作符都可以从上一个操作符中获得Observable。它对其进行修改并创建一个新的Observable，该Observable成为下一个Observable的输入。

以下示例显示了filer和map运算符使用管道链接到一起，filter运算符删除所有小于等于2的数据，map运算符将该值乘以2。
输入流为[1，2，3，4，5]，而输出流为[6，8，10]。

~~~ts
obs.pipe(
 obs = new Observable((observer) => {
    observer.next(1)
    observer.next(2)
    observer.next(3)
    observer.next(4)
    observer.next(5)
    observer.complete()
  }).pipe(
    filter(data => data > 2),   //filter Operator
    map((val) => {return val as number * 2}), //map operator
  )
 ~~~

 # 取消订阅Observable

当我们不再需要observable时，我们需要取消订阅以关闭它。如果不取消订阅，可能会导致内存泄漏和性能下降。
要从可观察到的订阅中取消订阅，我们需要对订阅调用Unsubscribe（）方法。它将清理所有侦听器并释放内存。要做到这一点，首先，创建一个变量来存储订阅。

~~~ts
obs: Subscription;
~~~

将订阅分配给obs变量

~~~ts
this.obs = this.src.subscribe(value => {
  console.log("Received " + this.id);
});
~~~

在ngOnDestroy方法中调用unsubscribe()方法。

~~~ts
ngOnDestroy() {
  this.obs.unsubscribe();
}
~~~
当我们销毁组件时，可观察到的内容将被取消订阅并清除。但是，您不必每次订阅都取消订阅。例如，发出完整信号的可观测对象，自动关闭。

# 总结

响应式编程就是对流进行编程。RxJS库将响应式编程引入了Angular。使用RxJs，我们可以创建一个可观测对象，它可以向观测对象的订阅者发出值、错误和完成信号。

