---
title: 使用Renderer2操作DOM元素
date: 2023-04-06 15:13:47
tags:
---
Renderer2允许我们在不直接访问DOM的情况下操作DOM元素。它在DOM元素和组件代码之间提供了一层抽象。使用Renderer2，我们可以创建一个元素，向其中添加一个文本节点，使用appendchild方法附加子元素。我们还可以添加或删除样式、HTML属性、CSS类和属性等。我们还可以附加和侦听DOM事件。

# 为什么不直接使用 ElementRef

ElelemtRef的nativeElement属性包含对底层DOM对象的引用，这使我们可以绕过Angular直接访问DOM，我们可以使用nativeElement属性来直接操作DOM元素。但由于以下原因，直接操作DOM是不建议的。

1. Angular使用模板、数据绑定和更改检测等保持组件和视图同步。当我们直接更新DOM时，所有这些都会被绕过。
2. DOM操作仅适用于浏览器。您将无法在其他平台中使用该应用程序，例如在服务器（服务器端渲染）、桌面或移动应用程序等中。
3. DOM API不会对数据进行净化。因此，可以通过注入脚本，打开我们的应用程序，成为XSS注入攻击的目标。

# 使用 Renderer2

首先从  @angular/core 导入 Renderer2.

~~~ts
import {Component, Renderer2, ElementRef, ViewChild, AfterViewInit } from '@angular/core';
~~~

在组件中注入 Renderer2

~~~ts
constructor(private renderer:Renderer2) {
}
~~~
使用 ElementRef & ViewChild 获取我们想操作的DOM元素实例

~~~ts
@ViewChild('hello', { static: false }) divHello: ElementRef;
~~~

使用setProperty、setStyle等方法更改元素的属性和样式，如下所示
~~~ts
this.renderer.setProperty(this.divHello.nativeElement,'innerHTML',"Hello Angular")
 
this.renderer.setStyle(this.divHello.nativeElement, 'color', 'red');
~~~

# 设置&移除 Styles

使用setStyle和RemoveStyle添加或删除样式，它接受四个参数。第一个参数是我们要应用样式的元素，第二个参数是样式的名称，第三个参数是样式的值，最后一个参数是可选的，他是样式变量的标志
~~~ts
abstract setStyle(el: any, style: string, value: any, flags?: RendererStyleFlags2): void
 
abstract removeStyle(el: any, style: string, flags?: RendererStyleFlags2): void
~~~

例如：
~~~ts
//Template
<div #hello>Hello !</div>
 
//Component
@ViewChild('hello', { static: false }) divHello: ElementRef;
 
setStyle() {
  this.renderer.setStyle(this.divHello.nativeElement, 'color', 'blue');
}
 
removeStyle() {
  this.renderer.removeStyle(this.divHello.nativeElement, 'color');
}
~~~

使用最后一个选项RendererStyleFlags2指定渲染器特有样式修饰符

# 添加&删除CSS 样式

使用 addClass / removeClass 添加&删除CSS样式。

~~~ts
abstract addClass(el: any, name: string): void
 
abstract removeClass(el: any, name: string): void
~~~

例如：
~~~ts
//Template
<div #hello>Hello !</div>

//Component
@ViewChild('hello', { static: false }) divHello: ElementRef;
 
addClass() {
  this.renderer.addClass(this.divHello.nativeElement, 'blackborder' );
}
 
removeClass() {
  this.renderer.removeClass(this.divHello.nativeElement, 'blackborder');
}
~~~

# 添加删除 Attributes 

使用 setAttribute & removeAttribute 添加&移除 attribute样式。

> 元素没有此 attribute，也可以添加上去
> attribute，还可以做动词，表示赋予，属于人为赋予的可改变的属性
~~~ts
setAttribute(el: any, name: string, value: string, namespace?: string): void
 
removeAttribute(el: any, name: string, namespace?: string): void
~~~

# 设置Property
使用setProperty方法设置DOM元素的任何属性。

> 元素没有此 property 不会添加上去
> property是 物体本身自带属性，不能改变的（一旦改了就是另外一个东西了）

~~~ts
setProperty(el: any, name: string, value: any): void
~~~

# AppendChild createElement createText
使用appendChild将一个新元素（子元素）附加到任何现有元素（父元素）。

~~~ts
appendChild(parent: any, newChild: any): void
~~~
它接受两个参数。第一个参数是父节点(我们希望在其中附加一个新的子节点)。第二个参数是要添加的子节点。

## 创建一个新元素

~~~ts
const div = this.renderer.createElement('div');

const text = this.renderer.createText('Inserted at bottom');

this.renderer.appendChild(div, text);

this.renderer.appendChild(this.div.nativeElement, div);

~~~

# InsertBefore
我们还可以使用insertBefore方法在DOM元素中的元素之前插入新元素。insertBefore的语法如下所示
~~~ts
insertBefore(parent: any, newChild: any, refChild: any): void
~~~
parent是父节点,newChild是要插入的新节点,refChild是插入newChild之前的现有子节点。

# 添加注释

createComment创建注释节点。它接受注释作为自变量。然后可以使用appendChild或insertBefore将其插入DOM中的任何位置。
~~~ts
createComment(value: string): any
~~~

# ParentNode & NextSibling

ParentNode方法返回宿主元素DOM中给定节点的父节点。

~~~ts
/Returns the parent Node of div3
this.renderer.parentNode(this.div3.nativeElement);
~~~
nextSibling方法返回宿主元素DOM中给定节点的下一个同级节点。

~~~ts
//Returns the next Sibling node of div2
this.renderer.nextSibling(this.div2.nativeElement);
~~~

# SelectRootElement

我们也可以使用selectRoomElement来选择基于选择器的节点元素。

~~~ts
selectRootElement(selectorOrNode: any, preserveContent?: boolean)
~~~
第一个参数是选择器或节点。Renderer2使用选择器来搜索DOM元素并返回它。

第二个参数是preserveContent，如果是no或undefined，renderer2将删除所有子节点。如果是yes，则不会删除子节点。

# 监听 DOM 事件

您也可以使用listen方法来侦听DOM事件。

listen方法接受三个参数，第一个参数是DOM元素（目标），第二个参数是事件的名称（eventName），第三个参数是回调函数

~~~ts
abstract listen(target: any, eventName: string, callback: (event: any) => boolean | void): () => void
~~~
