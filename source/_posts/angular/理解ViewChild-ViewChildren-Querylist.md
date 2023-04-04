---
title: 理解ViewChild, ViewChildren & Querylist
date: 2023-04-04 14:03:01
tags:
---
ViewChild或ViewChildren装饰器用于查询和获取组件中DOM元素的引用。ViewChild返回第一个匹配的元素，ViewChildren以QueryList形式返回所有匹配的元素。我们可以通过这些引用在组件中操作dom元素属性。

ViewChild或ViewChildren的第一个参数是查询选择器，通过ViewChild或ViewChildren来查询DOM元素，我们可以提供字符串或类型作为查询选择器。参数static确定是在更改检测之前还是之后执行查询。read选项允许我们查询不同的令牌，而不是默认令牌，并且在元素与多个类型关联时很有用。

# ViewChild
ViewChild查询从DOM返回第一个匹配元素，并更新它在组件中绑定的变量。

## 语法

~~~ts
ViewChild(selector: string | Function | Type<any>, opts: { read?: any; static: boolean; }): any
~~~
我们在组件属性上应用viewChild装饰器。它需要提供两个参数，selector和opts。

+ selector：可以是字符串、类型或返回字符串或类型的函数。变更检测查找与选择器匹配的第一个元素，并使用对该元素的引用更新组件属性。如果DOM发生更改，并且有一个新元素与选择器匹配，则变更检测会更新组件属性。

+ opts：有两个选项。

    + static：确定何时解析查询。设为True时，当视图首次初始化（在第一次更改检测之前）。设为False时，在每次检测到更改后执行。
    + read：使用它从查询的元素中读取不同的令牌。
# 例子

## 在组件和指令中注入引用

ViewChild的一个用例是在父组件中获取子组件的引用并操作其属性。

~~~ts
import { Component } from '@angular/core';
 
@Component({
  selector: 'child-component',
  template: `<h2>Child Component</h2>
            current count is {{ count }}
    `
})
export class ChildComponent {
  count = 0;
 
  increment() {
    this.count++;
  }
  decrement() {
    this.count--;
  }
}
~~~

我们可以在父组件中使用 ViewChild 引用子组件。

~~~ts
@ViewChild(ChildComponent, {static:true}) child: ChildComponent;
~~~
在上面的代码中，ViewChild在父组件视图(Template)中查找第一个出现的ChildComponent组件，并更新父组件中的child变量。现在我们可以从父组件调用ChildComponent组件中的Increment和Decrement方法。

~~~ts
import { Component, ViewChild } from '@angular/core';
import { ChildComponent } from './child.component';
 
@Component({
  selector: 'app-root',
  template: `
        <h1>{{title}}</h1>
        <p> current count is {{child.count}} </p>
        <button (click)="increment()">Increment</button>
        <button (click)="decrement()">decrement</button>
        <child-component></child-component>
        ` ,
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'Parent calls an @ViewChild()';
  
  @ViewChild(ChildComponent, {static:true}) child: ChildComponent;
 
  increment() {
    this.child.increment();
  }
 
  decrement() {
    this.child.decrement();
  }
 
}
~~~

## 使用模板引用变量

您可以使用模板引用变量代替组件类型。

~~~html
<child-component #child></child-component>
~~~

然后在ViewChild查询中使用它来获取对该组件的引用。

~~~ts
 @ViewChild("child", { static: true }) child: ChildComponent;
~~~

## 使用ElementRef注入HTML元素

Viewchild 同样可以用来查询 HTML元素。
首先可以给HTML元素指定模板变量，然后就可以在viewChild中使用此模板变量查询HTML元素。查询结果将返回一个ElementRef类型的元素，它是本地HTML元素的包装器。

~~~ts
import {AfterViewInit, Component, ElementRef, ViewChild} from '@angular/core';
 
@Component({
    selector: 'htmlelement',
    template: `
      <p #para>Some text</p>
    `,
})
export class HTMLElementComponent implements AfterViewInit {
 
    @ViewChild('para',{static:false}) para: ElementRef;
 
    ngAfterViewInit() {
      console.log(this.para.nativeElement.innerHTML);
      this.para.nativeElement.innerHTML="new text"
    }
}
~~~

## 多个实例
模板中可能存在同一组件或元素的多个实例。

~~~html
<child-component></child-component>
<child-component></child-component>
~~~
ViewChild 总是返回第一个匹配的组件。
~~~ts
@ViewChild(ChildComponent, {static:true}) child: ChildComponent;
~~~
要获取子组件的所有实例，我们可以使用ViewChildren，我们将在本教程后面介绍它。

## ViewChild 返回 Undefined

ViewChild返回Undefined是我们在使用它们时遇到的常见错误之一。

该错误是由于我们试图在ViewChild初始化之前使用该值。

例如，下面的代码导致无法读取未定义的属性“increment”。当构造函数运行前，组件的视图尚未初始化。因此，Angular无法通过引用ChildComposet变量来更新child变量。

~~~ts
export class AppComponent {
  title = 'Parent calls an @ViewChild()';
  
  @ViewChild(ChildComponent, {static:true}) child: ChildComponent;
 
  constructor() {
    this.child.increment()
  } 
 
}
 
//
Cannot read property 'increment' of undefined
~~~
解决方案是等待Angular初始化视图。Angular在完成视图初始化后会引发AfterViewInit生命周期挂钩。因此，我们可以使用ngAfterViewInit来访问子变量。

~~~ts
ngAfterViewInit() {
    this.child.increment()
  }
~~~
现在，代码没有给出任何错误。

上面的代码也将与ngOnInit生命周期挂钩一起使用。但它不能保证一直工作，因为Angular可能不会在引发ngOnInit钩子之前初始化视图的所有部分。因此，最好使用ngAfterViewInit钩子。

此外，ViewChild更新值的时间也取决于静态选项

## 在ViewChild中使用Static选项

我们在上面的代码中使用了{static:true}。

static选项确定ViewChild查询解析的时间。

+ static:true 将在每次变更改检测之前解析ViewChild。

+ static:false 将在每次变更改检测之后解析ViewChild。

当动态渲染子对象时，static的值变得很重要。例如在ngIf或ngSwitch等内部。

例如，考虑下面的代码，我们子组件放在ngIf中

~~~html
//child.component.html
 
<h1>ViewChild Example</h1>
 
<input type="checkbox" id="showCounter" name="showCounter" [(ngModel)]="showCounter">
 
<ng-container  *ngIf="showCounter">
 
  <p> current count is {{child?.count}} </p>
 
  <button (click)="increment()">Increment</button>
  <button (click)="decrement()">decrement</button>
 
  <child-component></child-component>
 
</ng-container>
~~~

~~~ts
//child.component.ts
 
import { Component, ViewChild, AfterViewInit, OnInit, ChangeDetectorRef } from '@angular/core';
import { ChildComponent } from './child.component';
 
@Component({
  selector: 'app-root',
  templateUrl: 'app.component.html' ,
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'ViewChild Example)';
 
  showCounter: boolean = true
 
  @ViewChild(ChildComponent, { static: true }) child: ChildComponent;
 
  increment() {
    this.child.increment();
  }
 
  decrement() {
    this.child.decrement();
  }
 
}
~~~

上述代码导致TypeError:无法读取未定义的属性“increment”。即使我们将true赋值给showCounter，也会发生错误。
因为在上述情况下，Angular不会立即渲染子组件。但在第一次变更检测之后，angular会检测showCounter的值并渲染子组件。

由于我们使用了static:true，angular将在运行第一次更改检测之前尝试解析ViewChild。因此，child变量总是未定义的。
现在，更改static：false。现在，代码将正常工作。也就是说，因为在每次检测到更改之后，Angular都会更新ViewChild。

## 在 ViewChild 中使用 Read选项

单个元素可以与多种类型相关联。

例如，考虑以下代码#nameInput模板变量现在与input和ngModel都关联
~~~html
<input #nameInput [(ngModel)]="name">
~~~
下面的viewChild代码将input元素的实例作为elementRef返回。

~~~ts
 @ViewChild('nameInput',{static:false}) nameVar;
~~~

如果我们想获得ngModel的实例，那么我们使用Read选项指定需要的令牌类型。

~~~ts
@ViewChild('nameInput',{static:false, read: NgModel}) inRef;
@ViewChild('nameInput',{static:false, read: ElementRef}) elRef;
@ViewChild('nameInput', {static:false, read: ViewContainerRef }) vcRef;
~~~

Angular中的每个元素总是有一个ElementRef和ViewContainerRef与之关联。如果该元素是一个组件或指令，那么总是有组件或指令实例。您也可以对一个元素应用多个指令。
不带read令牌的ViewChild默认返回值类型为组件，如果返回值不是组件类型则返回elementRef类型。

## 从子组件注入提供程序

您还可以注入子组件中提供的服务。

~~~ts
import { ViewChild, Component } from '@angular/core';
 
@Component({
  selector: 'app-child',
  template: `<h1>Child With Provider</h1>`,
  providers: [{ provide: 'Token', useValue: 'Value' }]
})
 
export class ChildComponent{
}
~~~
在父组件中，可以使用read属性访问提供程序

~~~ts
import { ViewChild, Component } from '@angular/core';
 
@Component({
  selector: 'app-root',
  template: `<app-child></app-child>`,
})
 
export class AppComponent{
    @ViewChild(ChildComponent , { read:'Token', static:false } ) childToken: string;
}
~~~

