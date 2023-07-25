---
title: 从Angular中的字符串、数组和对象创建可观察对象
date: 2023-04-25 17:03:43
tags:rxjs
---

# Observable 的创建功能

有很多方法可以在Angular中创建Observable对象。您可以使用Observable构造函数，如Observable教程中所示。还有许多函数可以用来创建新的Observable。这些函数帮助我们从数组、字符串、promise等创建Observable。

+ create
+ defer
+ empty
+ from
+ fromEvent
+ interval
+ of
+ range
+ throw
+ timer
所有与创建相关的操作符都是RxJs核心库的一部分。您可以从“rxjs”库导入它

# Create

调用Create方法是创建Observable最简单的方式。Create是Observable对象的内容方法，所以您不必导入它。

~~~ts
ngOnInit() {
 
   //Observable from Create Method
   const obsUsingCreate = Observable.create( observer => {
     observer.next( '1' )
     observer.next( '2' )
     observer.next( '3' )
     observer.complete()
   })
    obsUsingCreate
      .subscribe(val => console.log(val),
              error=> console.log("error"),
              () => console.log("complete"))
}
****Output *****
1
2
3
Complete
~~~

## Observable 构造函数

我们在上一个示例中可以看到这一点，Observable.create方法和Observable构造函数之间没有区别。Observable的Create方法实际在幕后调用Observable的构造函数。

~~~ts
ngOnInit() {
   //Observable Using Constructor
   const obsUsingConstructor = new Observable( observer => {
      observer.next( '1' )
      observer.next( '2' )
      observer.next( '3' )
 
      observer.complete()
   })
 
   obsUsingConstructor
        .subscribe(val => console.log(val),
                error=> console.log("error"),
                () => console.log("complete"))
}
 
 
****Output *****
1
2
3
complete
~~~

# Of 操作符 Of(a,b,c)

~~~ts
ngOnInit() {
  const array=[1,2,3,4,5,6,7]
  const obsof1=of(array);
  obsof1.subscribe(val => console.log(val),
           error=> console.log("error"),
          () => console.log("complete"))
 
}
**** Output ***
[1, 2, 3, 4, 5, 6, 7]
complete
~~~

# From 操作符 from([1,2,3])
From 操作符受一个可以迭代的参数，并将其转换为Observable的参数。

~~~ts
ngOnInit() {
 
    const array3 = [1, 2, 3, 4, 5, 6, 7]
    const obsfrom1 = from(array3);
    obsfrom1.subscribe(val => console.log(val),
      error => console.log("error"),
      () => console.log("complete"))
}
*** Output ****
1
2
3
4
5
6
7
complete

ngOnInit() { 
  const obsfrom2 = from('Hello World');
    obsfrom2.subscribe(val => console.log(val),
      error => console.log("error"),
      () => console.log("complete"))
}
*** Output ****
H
e
l
l
o
 
W
o
r
l
d
complete

~~~

# Of Vs From
|Of|From|
|-|-|
|接受变量(并不是参数)|只接受一个参数|
|在不更改任何内容的情况下按原样发出每个变量|迭代参数并发出每个值|

# fromEvent 操作符

## 语法

~~~ts
fromEvent<T>(target: FromEventTarget<T>, 
             eventName: string, 
             options: EventListenerOptions, 
             resultSelector: (...args: any[]) => T): Observable<T>
~~~

FromEventsTarget：是fromvevent的第一个参数。它可以是DOM EventTarget、Node.js EventEmitter、类似JQuery的事件目标、NodeList或HTMLCollection。目标必须有一个方法来注册/注销事件处理程序。（如果是DOM事件目标，则为addEventListener/removeEventListener）

eventName：是第二个参数，这是我们想要侦听的事件类型。

eventListenerOptions：是我们在注册事件处理程序（即addEventListener）时要传递给的附加参数

resultSelector：是可选的，在未来的版本中将不推荐使用

## 示例

~~~ts

`<button #btn>Button</button>
`
@ViewChild('btn', { static: true }) button: ElementRef;

 buttonClick() {
    this.buttonSubscription =  fromEvent(this.button.nativeElement, 'click')
        .subscribe(res => console.log(res));
  }

ngAfterViewInit() {
    this.buttonClick();
  }


~~~
我们可以从ngAfterViewInit方法调用上述方法。请注意，@ViewChild在ngOnInit之前不会初始化btn元素。因此，我们在这里使用ngAfterViewInit。

# pipe 操作符

Angular Observable的管道方法用于将多个操作符链接在一起。我们可以将管道作为一个独立的方法使用，这有助于我们在多个地方重用它或将其作为一个实例方法。在本教程中，我们将了解管道，并了解如何在Angular应用程序中使用它。我们将向您展示使用map、filter和tap操作符的管道示例。

## 使用Pipe 链接操作符

管道方法接受filter、map等运算符作为参数。每个参数必须用逗号分隔。运算符的顺序很重要，因为当用户订阅可观察对象时，管道会按添加运算符的顺序执行运算符。

我们有两种方法可以使用这个管道。一种是作为observable的实例，另一种是作为独立方法使用

## pipe作为实例方法

管道作为实例方法如下所示。我们将运算符op1、op2等作为参数传递给管道方法。op1方法的输出变成op2运算符的输入，依此类推。
~~~ts
obs.pipe(
  op1(),
  op2(),
  op3(),
  op3(),
)
~~~
以下是将管道与map & filter操作符一起使用的示例。

~~~ts
import { Component, OnInit } from '@angular/core';
import { Observable, of} from 'rxjs';
import { map, filter, tap } from 'rxjs/operators'
 
 
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
 
  obs = new Observable((observer) => {
    observer.next(1)
    observer.next(2)
    observer.next(3)
    observer.next(4)
    observer.next(5)
    observer.complete()
  }).pipe(
    filter(data => data > 2),                    //filter Operator
    map((val) => {return val as number * 2}),    //map operator
  )
 
 data = [];
 
  ngOnInit() {
    this.obs1.subscribe(
      val => {
        console.log(this.data)
      }
    )
  }
 
}
 
 
//result
[6, 8, 10]

import { Component, OnInit } from '@angular/core';
import { Observable, of, pipe } from 'rxjs';
import { map, filter, tap } from 'rxjs/operators'
 
 
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
 
  obs = new Observable((observer) => {
    observer.next(1)
    observer.next(2)
    observer.next(3)
    observer.next(4)
    observer.next(5)
    observer.complete()
  }).pipe(
    tap(data => console.log('tap '+data)),           //tap
    filter(data => data > 2),                        //filter
    tap(data => console.log('filter '+data)),        //tap
    map((val) => { return val as number * 2 }),      //map
    tap(data => console.log('final '+data)),         //tap
  )
 
 
  data = [];
 
  ngOnInit() {
 
    this.obs.subscribe(
      val => {
        this.data.push(val)
        console.log(this.data)
      }
    )
 
  }
}
~~~

## Pipe 作为单独方法

我们也可以将管道作为一个独立的函数来组成操作符，并在其他地方重用管道。

~~~ts
customOperator = pipe(
    tap(data => console.log('tap '+data)),
    filter(data => data > 2),
    tap(data => console.log('filter '+data)),
    map((val) => {
      return val as number * 2
    }),
    tap(data => console.log('final '+data)),
  );
 
 
  obs = new Observable((observer) => {
    observer.next(1)
    observer.next(2)
    observer.next(3)
    observer.next(4)
    observer.next(5)
    observer.complete()
  })
  
  ngOnInit() {
    this.customOperator(this.obs).subscribe();
~~~

~~~ts
  customOperator = pipe(
    tap(data => console.log('tap '+data)),
    filter(data => data > 2),
    tap(data => console.log('filter '+data)),
    map((val) => {
      return val as number * 2
    }),
    tap(data => console.log('final '+data)),
  );
 
 
  obs = new Observable((observer) => {
    observer.next(1)
    observer.next(2)
    observer.next(3)
    observer.next(4)
    observer.next(5)
    observer.complete()
  }).pipe(    
    this.customOperator,
    tap(data => console.log('final '+data)),
  )
 
 
  data = [];
 
  ngOnInit() {
 
    this.obs.subscribe(
      val => {
        this.data.push(val)
        console.log(this.data)
      }
    )
 
  }
}
~~~



# 总结

我们可以使用Create方法或Observable构造函数来创建一个新的Observable。
当您有类似数组的值时，Of运算符很有用，您可以将其作为单独的参数传递给Of方法以创建可观察的值。
From操作符试图迭代传递给它的任何东西，并从中创建一个可观察的对象。
RxJS库中有许多其他运算符或方法可用于创建和操作Angular observable。我们将在接下来的几个教程中学习其中的一些内容
