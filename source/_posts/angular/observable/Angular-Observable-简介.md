---
title: Angular Observable 简介
date: 2023-04-14 11:51:36
tags:rxjs
---

# 什么是RxJS
RxJS（Reactive Extensions Library for JavaScript）是一个使用可观察序列编写异步和基于事件的JavaScript库，RxJS 属于响应式编程，其思想是将时间看作数组，随着时间发生的事件被看作是数组的项，然后以操作数组的方式变换事件。

**RxJS 中解决异步事件管理的基本概念有：**

+ **Observable（可观察者）**：它的本质其实就是一个随时间不断产生数据的一个集合，称之为流更容易理解。

+ **Observer（观察者）**：从行为上来看，无非就是定义了如何处理上述流产生的数据，称之为流的处理方法，更容易理解。

+ **Subscription（订阅）**：表示 Observable 的一次执行，它的本质就是暂存了一个启动后的流，每一个启动后的流都是相互独立的，而这个启动后的流，就存储在subscription中，提供了unsubscribe，来停止这个流。

+ **Operator（操作符）**：是纯函数，可以使用 map、filter、concat、reduce 等操作来以函数式编程风格处理集合。

+ **Subject（主体）**：相当于一个 EventEmitter，也是将一个值或事件多播到多个 Observers 的唯一方式。

+ **Scheduler（调度器）**：是控制并发的集中化调度器，允许我们在计算发生时进行协调，例如 setTimeout 或 requestAnimationFrame 或其它。


Angular在其框架中大量使用RxJS库来实现响应式编程。使用响应式编程的一些示例如下：

+ Angular中响应HTTP请求
+ Angular中表单中值/状态变更
+ Router和Forms模块使用可观察性来监听和响应用户输入事件
+ 通过自定义事件从子组件发送observable输出数据到父组件
+ HTTP模块使用Observable来处理AJAX请求和响应

那么流是指什么呢？举个例子，代码中每1s输出一个数字，用户每一次对元素的点击，就像是在时间这个维度上，产生了一个数据集。这个数据集不像数组那样，它不是一开始都存在的，而是随着时间的流逝，一个一个数据被输出出来。这种异步行为产生的数据，就可以被称之为一个流，在Rxjs中，称之为observable（抛开英文，本质其实就是一个数据的集合，只是这些数据不一定是一开始就设定好的，而是随着时间而不断产生的）。而Rxjs，就是为了处理这种流而产生的工具，比如流与流的合并，流的截断，延迟，消抖等等操作。

# 理解基本定义: observable, observer, subscription

```javascript
import { Observable } from "rxjs";

// 流的创建
const stream$ = new Observable(subscriber => {
  setTimeout(() => {
    subscriber.next([1, 2, 3]);
  }, 500);
});

// 如何消费流产生的数据，observer
const observer = {
  complete: () => console.log("done"),
  next: v => console.log(v),
  error: () => console.log("error")
};

// 启动流
const subscription = stream$.subscribe(observer);

setTimeout(() => {
  // 停止流
  subscription.unsubscribe();
}, 1000);
```
上述过程中，涉及到了3个变量 :

1. stream$, 对应到Rxjs中，就是一个observable，它的本质其实就是一个随时间不断产生数据的一个集合，称之为流更容易理解。而其对象存在一个subscribe方法，调用该方法后，才会启动这个流（也就是数据才会开始产生），这里需要注意的是多次启动的每一个流都是独立的，互不干扰。

2. observer，代码中是stream$.subscribe(observer)，对应到Rxjs中，也是称之为observer。从行为上来看，无非就是定义了如何处理上述流产生的数据，称之为流的处理方法，更容易理解。

3. subscription，也就是const subscription = stream$.subscribe(observer); 它的本质就是暂存了一个启动后的流，之前提到，每一个启动后的流都是相互独立的，而这个启动后的流，就存储在subscription中，提供了unsubscribe，来停止这个流。

简单理解了这三个名词observable, observer, subscription后，从数据的角度来思考：observable定义了要生成一个什么样的数据，其subscribe方法，接收一个observer（定义了接收到数据如何处理），并开始产生数据，该方法的返回值，subscription, 存储了这个已经开启的流，同时具有unscbscribe方法，可以将这个流停止。

整理成下面这个公式：

>Subscription = Observable.subscribe(observer)
> + observable: 随着时间产生的数据集合，可以理解为流，其subscribe方法可以启动该流
> + observer: 决定如何处理数据
> + subscription: 存储已经启动过的流，其unsubscribe方法可以停止该流

这里有几个点：
1. subscribe不是订阅，而是启动这个流，可以看到，subscribe后，才会执行next方法 
2.  构建observable的时候，会有一个subscriber.next，这里就是控制这个流中数据的输出。
3.  Observable流可以多次启动，多次启动的流之间是相互独立的。
4.  Observable.subscribe()的返回值subscription上存在一个方法unsubscribe，可以将流停止。


# 创建 Observable 对象

Observable 是个多值的惰性 Push 集合。他们填补了下表中的缺失点：
||单值|多值|
|-|-|-|
|拉|Function|Iterator|
|推|Promise|Observable|
|

## 创建 Observables
Observable 的构造函数接受一个参数：subscribe 函数。

下面的示例创建一个 Observable 以每秒向订阅者发送字符串 'hi'。
```javascript

import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  const id = setInterval(() => {
    subscriber.next('hi');
  }, 1000);
});
```
## 订阅 Observables
示例中的 Observable observable 可以被订阅，如下所示：

```javascript
observable.subscribe((x) => console.log(x));
```
subscribe 调用只是一个启动“ Observable 的执行”并将一些值或事件传递给该执行过程的 Observer 的方法。
同一个 Observable 的多个 Observer 之间是不共享的。对 observable.subscribe 的每次调用都会为给定的订阅者触发其自己的独立处理方法。

## 执行 Observables
在 Observable 执行中，可能会传递零个到无限个 Next 通知。如果发送了出错或完成通知，则之后将无法发送任何其它通知。

**Observable 执行可以传递三种类型的值：**

+ “Next（下一个）” 通知：发送数值、字符串、对象等。

+ “Error（出错）” 通知：发送 JavaScript 错误或异常。

+ “Complete（完成）”通知：不发送值。
```javascript
import { Observable } from 'rxjs';
 
const observable = new Observable(function subscribe(subscriber) {
  try {
    subscriber.next(1);
    subscriber.next(2);
    subscriber.next(3);
    subscriber.complete();
  } catch (err) {
    subscriber.error(err); // delivers an error if it caught one
  }
});
```
## 取消 Observable 执行
当 observable.subscribe 被调用时，此 Observer 被附加到新创建的 Observable 执行中。此调用还会返回一个对象 Subscription ：

```javascript
const subscription = observable.subscribe((x) => console.log(x));

```
当你订阅时，你会得到一个 Subscription，它代表正在进行的执行。只需调用 unsubscribe() 即可取消执行。
```javascript
import { from } from 'rxjs';

const observable = from([10, 20, 30]);
const subscription = observable.subscribe((x) => console.log(x));
// Later:
subscription.unsubscribe();
```

当我们不再需要observable时，我们需要取消订阅以关闭它。如果不取消订阅，可能会导致内存泄漏和性能下降。
要从可观察到的订阅中取消订阅，我们需要对订阅调用Unsubscribe（）方法。它将清理所有侦听器并释放内存。

当我们销毁组件时，可观察到的内容将被取消订阅并清除。但是，您不必每次订阅都取消订阅。例如，发出完整信号的可观测对象，自动关闭。

# 观察者（Observer）

什么是 Observer？ Observer 是 Observable 传递的各个值的消费者。 Observer 只是一组回调，对应于 Observable 传递的每种类型的通知：next、error 和 complete。下面是一个典型的 Observer 对象的例子：
```javascript
const observer = {
  next: (x) => console.log('Observer got a next value: ' + x),
  error: (err) => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```
要使用 Observer，请将其提供给 Observable 的 subscribe ：
```javascript
observable.subscribe(observer);
```
> Observer 只是具有三个回调的对象，分别用于 Observable 可能传递的每种类型的通知。

RxJS 中的 Observer 也可能是部分的。如果你不提供其中一个回调，Observable 的执行仍然会正常进行，除了某些类型的通知会被忽略，因为它们在 Observer 中没有对应的回调。

~~~ts
ngOnInit() {
 
    this.obs.subscribe(
      val => { console.log(val) }, //next callback
      error => { console.log("error") }, //error callback
      () => { console.log("Completed") } //complete callback
    )
}
~~~

  # Observable 操作符

操作符是对Observable进行操作并返回新Observable的函数。

**有两种操作符：**
+ 可联入管道的操作符：本质上是一个纯函数，它将一个 Observable 作为输入并生成另一个 Observable 作为输出。订阅此输出 Observable 也会同时订阅其输入 Observable。
+ 创建操作符：是另一种操作符，可以作为独立函数调用以创建新的 Observable。例如： of(1, 2, 3) 创建一个 observable，它将一个接一个地发出 1、2 和 3。

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

# 弹珠图
要解释操作符的工作原理，文字描述通常是不够的。许多操作符都与时间有关，他们可能以不同的方式延迟、采样、节流或防抖后发出。图表通常是更好的工具。弹珠图是操作符如何工作的可视化表示，包括输入 Observable、操作符及其参数以及输出 Observable。



# Subject

Subject 是一种特殊类型的 Observable，它允许将值多播到多个 Observer。
+ 每个 Subject 都是 Observable。给定一个 Subject，你可以 subscribe 它，提供一个 Observer，它将开始正常接收值。
+ 每个 Subject 也都是 Observer。它是一个具有方法 next(v)、error(e) 和 complete() 的对象。要为 Subject 提供一个新值，只需调用 next(theValue)，它将被多播到注册进来监听 Subject 的 Observer。

Subject也是Rxjs中比较重要的概念，从英文上不太好理解，直接上代码：

```javascript
import { Subject } from "rxjs";

// 创建subject
const subject = new Subject();

// 订阅一个observer
subject.subscribe(v => console.log("stream 1", v));
// 再订阅一个observer
subject.subscribe(v => console.log("stream 2", v));
// 延时1s再订阅一个observer
setTimeout(() => {
  subject.subscribe(v => console.log("stream 3", v));
}, 1000);
// 产生数据1
subject.next(1);
// 产生数据2
subject.next(2);
// 延时3s产生数据3
setTimeout(() => {
  subject.next(3);
}, 3000);
// output
// stream 1 1 //立即输出
// stream 2 1 //立即输出
// stream 1 2 //立即输出
// stream 2 2 //立即输出
// stream 1 3 //3s后输出
// stream 2 3 //3s后输出
// stream 3 3 //3s后输出
```
可以看到，Subject的行为和发布订阅模式非常接近，subscribe去订阅，next触发。事件的订阅通过subscribe，事件的触发使用next，从而实现一个发布订阅的模式。

由于 Subject 是 Observer，这也意味着你可以提供 Subject 作为任意 Observable subscribe 的参数，如下面的示例所示：
```javascript
import { Subject, from } from 'rxjs';
 
const subject = new Subject<number>();
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});
 
const observable = from([1, 2, 3]);
 
observable.subscribe(subject); // You can subscribe providing a Subject
 
// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```
使用上述方法，我们基本上只是通过 Subject 将单播 Observable 执行转换为多播。这展示了 Subjects 是让任何 Observable 执行共享给多个 Observers 的唯一方法。

##  BehaviorSubject
它存储发送给其消费者的最新值，并且每当有新的 Observer 订阅时，它将立即从 BehaviorSubject 接收到“当前值”。

在下面的示例中，BehaviorSubject 使用第一个 Observer 在订阅时收到的值 0 进行初始化。第二个 Observer 接收到值 2，即使它是在发送值 2 之后订阅的。
```javascript
import { BehaviorSubject } from 'rxjs';
const subject = new BehaviorSubject(0); // 0 is the initial value

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(3);

// Logs
// observerA: 0
// observerA: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```
## ReplaySubject
ReplaySubject 与 BehaviorSubject 类似，它可以将旧值发送给新订阅者，但它也可以记录 Observable 执行结果的一部分。

创建 ReplaySubject 时，你可以指定要重播的值的数量：
```javascript
import { ReplaySubject } from 'rxjs';
const subject = new ReplaySubject(3); // buffer 3 values for new subscribers

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(5);

// Logs:
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerB: 2
// observerB: 3
// observerB: 4
// observerA: 5
// observerB: 5
```

除了缓冲区大小之外，你还可以指定一个以毫秒为单位的窗口时间，以确定记录的值可以存在多长时间。在以下示例中，我们使用 100 个元素的大型缓冲区，但窗口时间参数仅为 500 毫秒。

```javascript
import { ReplaySubject } from 'rxjs';
const subject = new ReplaySubject(100, 500 /* windowTime */);

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

let i = 1;
setInterval(() => subject.next(i++), 200);

setTimeout(() => {
  subject.subscribe({
    next: (v) => console.log(`observerB: ${v}`),
  });
}, 1000);

// Logs
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerA: 5
// observerB: 3
// observerB: 4
// observerB: 5
// observerA: 6
// observerB: 6
// ...
```
## AsyncSubject
AsyncSubject 是一种变体，其中仅将 Observable 执行的最后一个值发送给其 Observer，并且仅在执行完成时发送。
```javascript
import { AsyncSubject } from 'rxjs';
const subject = new AsyncSubject();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(5);
subject.complete();

// Logs:
// observerA: 5
// observerB: 5
```

# 总结

响应式编程就是对流进行编程。RxJS库将响应式编程引入了Angular。使用RxJs，我们可以创建一个可观测对象，它可以向观测对象的订阅者发出值、错误和完成信号。

