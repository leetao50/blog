---
title: Angular中的ElementRef
date: 2023-04-05 22:04:06
tags:
---
Angular ElementRef是一个围绕原生DOM元素（HTML元素）对象的包装器。它包含属性nativeElement，该属性保存对底层DOM对象的引用。我们可以使用它来操作DOM。我们组件中使用ViewChild来获取模板HTML元素的ElementRef实例。Angular还在组件或指令的构造函数中注入宿主元素的ElementRef实例。在本教程中，让我们探讨如何使用ElementRef来获得HtmlElement的引用并在Angular Applications中操作DOM。

# ElementRef
DOM对象由浏览器创建和维护。它们代表了文件的结构和内容。在原生JavaScript代码中，我们访问这些DOM对象来操作View。我们可以创建文档，以及添加、修改或删除元素和内容。

Angular提供了许多工具和技术来操作DOM。我们可以添加/删除组件。它提供了许多指令，如类指令或样式指令，来操纵他们的风格等。

在某些情况下，我们可能仍然需要访问DOM元素。这就需要用到ElementRef。

# 在组件中获取ElementRef

要使用ElementRef操作DOM，我们需要在组件/指令中获取它对DOM元素的引用。

1. 获取对组件中DOM元素的引用

+ 为组件/指令中的元素创建模板引用变量。
+ 使用ViewChild或ViewChildren通过模板变量在组件中直接获取DOM元素引用。

2. 获取组件/指令的宿主DOM元素
+ 组件或指令的构造函数中注入宿主元素的ElementRef引用（Angular Dependency注入）

例如，在下面的代码中，变量hello引用了HTML元素div。
~~~html
<div #hello>Hello Angular</div>
~~~
我们可以在模板中使用hello这个模板引用变量。

在Component类中，我们使用ViewChild来注入hello元素。Angular将hello作为ElementRef类型注入。

~~~ts
@ViewChild('hello', { static: false }) divHello: ElementRef;
~~~

## 读取令牌

考虑以下示例
~~~html
<input #nameInput [(ngModel)]="name">
~~~

nameInput 模板引用变量现在绑定到input输入元素。但与此同时，我们也将ngModel指令绑定到它上面。

在这种情况下，我们可以使用read令牌让angular知道我们需要ElementRef引用到谁，如下所示

~~~ts
//ViewChild returns ElementRef i.e. input HTML Element
 
@ViewChild('nameInput',{static:false, read: ElementRef}) elRef;
 
 
//ViewChild returns NgModel associated with the nameInput
@ViewChild('nameInput',{static:false, read: NgModel}) inRef;
 
~~~

# ElementRef 例子

一旦我们有了ElementRef，我们就可以使用nativeElement属性来操作DOM，如下所示。
在访问ViewChild变量之前，我们需要等待Angular初始化视图。因此，我们要等到AfterViewInit生命周期挂钩之后，才能开始使用该变量。

~~~ts
import { Component,ElementRef, ViewChild, AfterViewInit } from '@angular/core';
 
@Component({
  selector: 'app-root',
  template:  '<div #hello>Hello</div>'
  styleUrls: ['./app.component.css']
})
export class AppComponent implements AfterViewInit {
 
 @ViewChild('hello', { static: false }) divHello: ElementRef;
 
 ngAfterViewInit() {
   this.divHello.nativeElement.innerHTML = "Hello Angular";
 }
 
}
~~~

我们可以非常容易得操作DOM元素
~~~ts
  ngAfterViewInit() {
    this.divHello.nativeElement.innerHTML = "Hello Angular";
    this.divHello.nativeElement.className="someClass";
    this.divHello.nativeElement.style.backgroundColor="red";
  }
~~~
# 在自定义指令中使用 ElementRef
ElementRef的一个用例是Angular指令。我们学习了如何在Angular中创建自定义指令。以下是ttClass自定义属性指令的代码。
~~~ts
import { Directive, ElementRef, Input, OnInit } from '@angular/core'
 
@Directive({
  selector: '[ttClass]',
})
export class ttClassDirective implements OnInit {
 
  @Input() ttClass: string;
 
  constructor(private el: ElementRef) {
  }
 
  ngOnInit() {
    this.el.nativeElement.classList.add(this.ttClass);
  }
 
}
~~~
请注意，我们在构造函数中注入ElementRef。每当我们访问构造函数中注入的ElementRef时，Angular就会注入对指令的宿主DOM元素的引用。

> 谨慎使用
> 当需要直接访问DOM时，使用此API作为最后的手段。请改用Angular提供的模板和数据绑定。或者，您可以看看Renderer2，它提供了即使不支持直接访问本机元素也可以安全使用的API。
> 依赖于直接DOM访问会在应用程序和渲染层之间产生紧密耦合，这将使您无法将两者分离并将应用程序部署到web工作者中。

# ElementRef和XSS注入攻击
ElementRef的不当使用可能导致XSS注入攻击。例如，在下面的代码中，我们正在使用elementRef注入一个脚本。当包含此类代码的组件运行时，将执行脚本
~~~ts
constructor(private elementRef: ElementRef) {
    const s = document.createElement('script');
    s.type = 'text/javascript';
    s.textContent = 'alert("Hello World")';
    this.elementRef.nativeElement.appendChild(s);
 }
 ~~~
