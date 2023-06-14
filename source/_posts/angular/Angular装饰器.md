---
title: Angular装饰器
date: 2023-04-13 15:29:52
tags:
---

我们使用装饰器为Angular中的类声明、方法、访问器、属性和参数提供元数据。我们使用它来装饰组件、指令、模块等。在本文中，让我们了解什么是装饰器，为什么需要它，以及如何创建自定义装饰器。我们还要了解Angular支持的内置装饰器。

# Angular 装饰器

Angular 装饰器是一个函数，我们使用它将元数据附加到类、方法、访问器、属性或参数。

我们以@expression形式使用装饰器，其中expression是装饰器的名称。

例如，@Component是一个装饰器，我们将其附加到一个Angular组件。当Angular看到@Component装饰器应用于一个类时，它将该类视为一个组件类。在下面的示例中，正是@Component装饰器使AppComponent成为一个Angular组件。如果没有装饰器，AppComponent就像其他类一样。

~~~ts
@Component({
})
export class AppComponent {
  title = 'app';
} 
~~~

decorator装饰器是一个Typescript特性，它仍然不是Javascript的一部分。它仍处于提案阶段。

要启用Angular 装饰器，我们需要将experialDecorators添加到tsconfig.json文件中。ng-new命令会自动为我们添加此内容。

~~~json

{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true
  }
}
~~~

# 创建新的装饰器
在下面的例子中，我们创建了一个函数simpleDecorator。我们将使用它来装饰AppComponent类。该函数不接受任何参数。

~~~ts
import { Component, VERSION } from '@angular/core';
 
@simpleDecorator      
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  name = 'Angular ' + VERSION.major;
 
  constructor() {
    console.log('Hello from Class constructor');
  }
 
  ngOnInit() {
    console.log((this as any).value1);
    console.log((this as any).value2);
  }
}
 
function simpleDecorator(target: any) {
  console.log('Hello from Decorator');
 
  Object.defineProperty(target.prototype, 'value1', {
    value: 100,
    writable: false
  });
 
  Object.defineProperty(target.prototype, 'value2', {
    value: 200,
    writable: false
  });
}
 
 
 
**** Console ***
 
Hello from Decorator
Hello from Class constructor
100
200
~~~

正如我们前面所说，d装饰器 是一个常规的JavaScript函数。由于我们在类上使用它，因此它获取AppComponent的实例作为参数（目标）

~~~ts

function simpleDecorator(target: any) {
  console.log('Hello from Decorator');
 
//target is instance of AppComponent
~~~

在函数内部，我们将两个自定义属性value1和value2添加到AppComponent。请注意，我们使用defineProperty属性向组件类添加一个新属性。此外，我们将其添加到类的原型属性中。

~~~ts
Object.defineProperty(target.prototype, 'value1', {
    value: 100,
    writable: false
  });
 
  Object.defineProperty(target.prototype, 'value2', {
    value: 200,
    writable: false
  });
 
 ~~~
现在，我们可以使用它来装饰我们的AppComponent

~~~ts
@simpleDecorator
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
 ~~~
在组件内部，我们可以使用关键字“this”访问新的属性。

~~~ts
ngOnInit() {
    console.log((this as any).value1);
    console.log((this as any).value2);
  }
~~~

# 带参数的装饰器

要创建一个带参数的装饰器，我们需要创建一个工厂函数，该函数返回装饰器函数。

~~~ts
import { Component, VERSION } from '@angular/core';
 
@simpleDecorator({
  value1: '100',
  value2: '200'
})
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  name = 'Angular ' + VERSION.major;
 
  constructor() {
    console.log('Hello from Class constructor');
  }
 
  ngOnInit() {
    console.log((this as any).value1);
    console.log((this as any).value2);
  }
}
 
function simpleDecorator(args) {
  console.log(args);
 
  return function(target: any) {
    console.log('Hello from Decorator');
    console.log(typeof target);
    console.log(target);
 
    Object.defineProperty(target.prototype, 'value1', {
      value: args.value1,
      writable: false
    });
 
    Object.defineProperty(target.prototype, 'value2', {
      value: args.value2,
      writable: false
    });
  };
}
 ~~~
 simpleDecorator将args作为参数，并返回 装饰器函数。除了使用参数填充财产之外，其余代码与上面的代码相同。

~~~ts
function simpleDecorator(args) {
  console.log(args);
 
  return function(target: any) {
 ~~~

 我们在组件上应用simpleDecorator，如下所示

# Angular 装饰器 列表

Angular提供了几个内置的装饰器。我们可以把它们分为四类

以下是Angular中装饰器的完整列表。

+ 类装饰器
  + @NgModule
  + @Component
  + @Injectable
  + @Directive
  + @Pipe
+ 属性装饰器
  + @Input
  + @Output
  + @HostBinding
  + @ContentChild
  + @ContentChildren
  + @ViewChild
  + @ViewChildren
+ 方法装饰器
  + @HostListener
+ 参数装饰器
  + @Inject
  + @Self
  + @SkipSelf
  + @Optional
  + @Host
  
# 类装饰器


