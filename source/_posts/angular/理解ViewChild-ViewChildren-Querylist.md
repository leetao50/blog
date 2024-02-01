---
title: 理解ViewChild, ViewChildren & Querylist
date: 2023-04-04 14:03:01
tags:
---
ViewChild或ViewChildren装饰器用于查询和获取组件中DOM元素的引用。ViewChild返回第一个匹配的元素，ViewChildren以QueryList形式返回所有匹配的元素。

# ViewChild
ViewChild查询从DOM返回第一个匹配元素，并更新它在组件中绑定的变量。

## 语法

~~~ts
ViewChild(selector: string | Function | Type<any>, opts: { read?: any; static: boolean; }): any
~~~
我们在组件属性上应用viewChild装饰器。它需要提供两个参数，selector和opts。

+ selector(查询选择器)：可以是字符串、类型或返回字符串或类型的函数。变更检测查找与选择器匹配的第一个元素，并使用对该元素的引用更新组件属性。如果DOM发生更改，并且有一个新元素与选择器匹配，则变更检测会更新组件属性。

+ opts：有两个选项。

    + static：选项确定ViewChild查询解析的时间。
      + static:true 将在每次变更改检测之前解析ViewChild(对象静态生成时)。
      + static:false 将在每次变更改检测之后解析ViewChild(对象动态渲染时)。
  
    + read：使用它从查询的元素中读取不同的令牌。

## 各类selector例子

### 通过class(组件和指令)

这个class也是有条件的，必须是@Component或者@Directive修饰的clas。

通过@ViewChild在父组件中获取子组件的引用并操作其属性。

~~~ts
import { Component } from '@angular/core';
 
@Component({
  selector: 'child-component',
  template: `<h2>Child Component</h2>
            current count is {{ count }}`
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

我们可以在父组件中使用@ViewChild引用子组件。

~~~ts
@ViewChild(ChildComponent, {static:true}) child: ChildComponent;
~~~
在上面的代码中，@ViewChild在父组件的视图中，查找第一个出现的ChildComponent组件，并更新父组件中的child变量。现在我们可以从父组件调用子组件(ChildComponent)中的Increment和Decrement方法。

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

### 通过子组件provider提供的类

比如我们有一个子组件。代码如下
```javascript
import {Component, OnInit} from '@angular/core';
import {ChildService} from './child.service';
@Component({
  selector: 'app-child',
  template: `
    <h1>自定义的一个子组件</h1>`,
  providers: [
    ChildService
  ]
})
export class ChildComponent implements OnInit {
  constructor(public childService: ChildService) {
  }
  ngOnInit() {
  }
}

```
 在上面的子组件里面provider里面提供了一个ChildService类。我们也是可以通过@ViewChild来拿到这个ChildService类的。代码如下
```ts
@ViewChild(ChildService) childService: ChildService;
```

### 子组件provider通过string token提供的类
子组件的pprovider通过 string token valued的形式提供了一个StringTokenValue类，string token 对应的是tokenService。

```javascript
import {Component} from '@angular/core';
import {StringTokenValue} from './string-token-value';
@Component({
  selector: 'app-child',
  template: `
    <h1>自定义的一个子组件</h1>`,
  styleUrls: ['./child.component.less'],
  providers: [
    {provide: 'tokenService', useClass: StringTokenValue}
  ]
})
export class ChildComponent {
}
```
在父组件里面我们也是可以拿到子组件provider的这个StringTokenValue类的。方式如下：

  ```ts
@ViewChild('tokenService') tokenService: StringTokenValue;

```


### 从子组件注入提供程序

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



### 通过模板变量

例如：
~~~html
<child-component #child></child-component>
~~~
里面的child就是模板变量。

在ViewChild查询中使用它来获取对该组件的引用。

~~~ts
 @ViewChild("child", { static: true }) child: ChildComponent;
~~~

~~~ts
import {AfterViewInit, Component, ElementRef, ViewChild} from '@angular/core';
 
@Component({
    selector: 'htmlelement',
    template: `
      <p #child>Some text</p>
    `,
})
export class HTMLElementComponent implements AfterViewInit {
 
    @ViewChild('child',{static:false}) para: ElementRef;
 
    ngAfterViewInit() {
      console.log(this.para.nativeElement.innerHTML);
      this.para.nativeElement.innerHTML="new text"
    }
}
~~~

### 通过 TemplateRef 
 当选择器是TemplateRef的时候，则会获取到html里面所有的ng-template的节点。实际例子如下：
```javascript
  /**** @ViewChild(TemplateRef) @ViewChildren(TemplateRef)获取页面上的ng-template节点信息 ****/
  @ViewChild(TemplateRef) template: TemplateRef<any>;
  @ViewChildren(TemplateRef) templateList: QueryList<TemplateRef<any>>;

```

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

+ static:true 将在每次变更改检测之前解析ViewChild(对象静态生成时)。

+ static:false 将在每次变更改检测之后解析ViewChild(对象动态渲染时)。


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

# ViewChildren

ViewChildren装饰器用于从View中获取元素引用的列表。ViewChildren与ViewChild不同。ViewChild始终返回对单个元素的引用。如果存在多个元素，则ViewChild返回第一个匹配元素，ViewChildren总是以QueryList的形式返回所有元素。您可以遍历列表并访问每个元素。

## 语法
viewChildren的语法如下所示。除了static选项之外，它与viewChild的语法非常相似。

```ts
ViewChildren(selector: string | Function | Type<any>, opts: { read?: any; }): any
```
> ViewChildren总是在运行更改检测之后解析。即为什么它没有静态选项。而且，您不能在ngOnInit钩子中引用它，因为它还没有初始化。

## QueryList

QueryList将viewChildren或contentChildren返回的项存储在列表中。
只要应用程序的状态发生变化，Angular就会更新此列表。它在每次检测到变化时都会这样做。
QueryList还实现了一个可迭代的接口。这意味着您可以使用for（var i of items）对其进行迭代，也可以在template*ngFor=“let i of item”中与ngFor一起使用。
可以通过订阅可观察的更改来观察更改。

您可以使用以下方法和属性。
+ first：返回列表中的第一个项目。
+ last：获取列表中的最后一项。
+ length：获取项目的长度。
+ changes：是可观察的。每当Angular添加、删除或移动子元素时，它都会发出一个新值。

## 实例
在下面的示例中，输入元素使用ngModel指令，我们使用ViewChildren来获取所有输入元素并存储在QueryList中。

最后，我们可以使用this.modelRefList.forEach循环查询列表并访问每个元素。
```javascript

import { ViewChild, Component, ViewChildren, QueryList, AfterViewInit } from '@angular/core';
import { NgModel } from '@angular/forms';
 
@Component({
  selector: 'app-viewchildren2',
  template: `
      <h1>ViewChildren Example</h1>
 
      <input *ngIf="showFirstName" name="firstName" [(ngModel)]="firstName">
      <input *ngIf="showMiddleName" name="midlleName" [(ngModel)]="middleName">
      <input *ngIf="showlastName" name="lastName" [(ngModel)]="lastName">
 
 
      <input type="checkbox" id="showFirstName" name="showFirstName" [(ngModel)]="showFirstName">
      <input type="checkbox" id="showMiddleName" name="showMiddleName" [(ngModel)]="showMiddleName">
      <input type="checkbox" id="showlastName" name="showlastName" [(ngModel)]="showlastName">
 
      <button (click)="show()">Show</button>`,
})
 
export class ViewChildrenExample2Component implements AfterViewInit {
 
  firstName;
  middleName;
  lastName;
 
  showFirstName=true;
  showMiddleName=true;
  showlastName=true;
 
  @ViewChildren(NgModel) modelRefList: QueryList<NgModel>;
 
  ngAfterViewInit() {
 
    this,this.modelRefList.changes
      .subscribe(data => {
        console.log(data)
      }
    )
  }
  
 
  show() {
    this.modelRefList.forEach(element => {
      console.log(element)
      //console.log(element.value)
    });
 
  }
}
```

# 指令
       @ViewChild、@ViewChildren也是可以获取到指令对象的。指令对象的获取和组件对象的获取差不多，唯一不同的地方就是用模板变量名获取指令的时候要做一些特殊的处理。我们还是用具体的实例来说明。我们自定义一个非常简单的指令TestDirective，添加exportAs属性。代码如下。

`exportAs属性很关键`
```javascript

import {Directive, ElementRef} from '@angular/core';

/**
 * 指令，测试使用,这里使用了exportAs，就是为了方便我们精确的找到指令
 */
@Directive({
  selector: '[appTestDirective]',
  exportAs: 'appTest'
})
export class TestDirective {

  constructor(private elementRef: ElementRef) {
    elementRef.nativeElement.value = '我添加了指令';
  }

}

```

获取TestDirective指令，注意单个指令对象获取的时候，模板变量名的写法。比如下面的代码中#divTestDirective='appTest'，模板变量名等号右边的就是TestDirective指令exportAs对应的名字。

import {AfterViewInit, Component, QueryList, ViewChild, ViewChildren} from '@angular/core';
import {TestDirective} from './test.directive';

```javascript
@Component({
  selector: 'app-root',
  template: `
    <!-- view child for Directive -->
    <input appTestDirective/>
    <br/>
    <input appTestDirective #divTestDirective='appTest'/>
  `,
  styleUrls: ['./app.component.less']
})
export class AppComponent implements AfterViewInit {
  /**
   * 获取html里面所有的TestDirective
   */
  @ViewChildren(TestDirective) testDirectiveList: QueryList<TestDirective>;
  /**
   * 获取模板变量名为divTestDirective的TestDirective的指令,这个得配合指令的exportAs使用
   */
  @ViewChild('divTestDirective') testDirective: TestDirective;

  ngAfterViewInit(): void {
    console.log(this.testDirective);
    if (this.testDirectiveList != null && this.testDirectiveList.length !== 0) {
      this.testDirectiveList.forEach(elementRef => console.log(elementRef));
    }
  }
}

```

 总结：@ViewChild、@ViewChildren获取子元素的的时候，我们用的最多的应该就是通过模板变量名，或者直接通过class来获取了。还有一个特别要注意的地方就是获取单个指令对象的时候需要配合指令的exportAs属性使用，并且把他赋值给模板变量名。