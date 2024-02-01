---
title: HostBinding & HostListener
date: 2023-04-03 17:10:50
tags:
---
HostBinding和HostListener是Angular中的装饰器。HostListener侦听宿主控件事件，而HostBinding允许我们绑定宿主控件的属性。

宿主控件是我们将要附加组件或指令到它上面的目标控件。此功能允许我们在用户对宿主控件执行某些操作或修改样式时采取一些措施；

# 宿主控件

宿主控件是我们附加指令或组件的元素。请记住，组件是带有视图（模板）的指令。

例如：

请考虑以下ttToggle指令。我们将其附加到按钮组件。这里的按钮组件是宿主组件。

~~~html
<button ttToggle>Click To Toggle</button>
~~~

在下面的例子中apphighlight是指令，p控件是宿主控件
~~~html
<div>
  <p apphighlight>
    <div>This text will be highlighted</div>
  </p>
</div>
~~~

# HostBinding

Host Binding将宿主控件属性绑定到指令或组件中的变量

例如：

以下appHighLight指令 通过 HostBinding 将父元素的style.border属性绑定到指令中的border属性。

因此我们更改指令中 border的值，angular将会更新父元素border样式。

~~~ts
import { Directive, HostBinding, OnInit } from '@angular/core'
 
@Directive({
  selector: '[appHighLight]',
})
export class HighLightDirective implements OnInit {
 
  @HostBinding('style.border') border: string;
 
  ngOnInit() {
    this.border="5px solid blue"
  }
 
}
~~~

~~~html
<div>
  <h2>appHighLight Directive</h2>
  <p appHighLight>
    This Text has blue Border
  </p>
</div>
~~~

# HostListener

HostListener 装饰器用来监听宿主元素上的DOM事件。它还提供了一个在事件发生时调用的处理程序。

例如，在以下代码中，HostListener侦听mouseover和mouseleave事件。我们使用HostListner在宿主元素的MouseOver和MouseLeave事件上绑定相应事件处理函数。

~~~ts
import { Directive, HostBinding, OnInit, HostListener } from '@angular/core'
 
@Directive({
  selector: '[appHighLight]',
})
export class HighLightDirective implements OnInit {
 
  @HostBinding('style.border') border: string;
 
  ngOnInit() {
  }
 
  @HostListener('mouseover') 
  onMouseOver() {
    this.border = '5px solid green';
    console.log("Mouse over")
  }
 
  @HostListener('mouseleave') 
  onMouseLeave() {
    this.border = '';
    console.log("Mouse Leave")
  }
 
}
~~~

我们在宿主元素p上面使用appHighLight指令，每当鼠标移动到p元素上时，鼠标悬停事件都会被HostListener捕获，它运将行我们附加到它的onMouseOver方法。该方法使用HostBinding为p元素添加一个绿色边界。

类似地，在mouseleave事件中，边框被移除。


~~~html
<div>
  <h2>appHighLight Directive</h2>
  <p appHighLight>
    This Text has blue Border
  </p>
</div>
~~~

# 使用HostBinding附加样式

将样式附加到宿主元素是HostBinding装饰器的常见用法之一。

例如，以下示例将highlight&box样式添加到宿主元素中

~~~ts
@HostBinding('class') class: string;
 
  ngOnInit() {
    this.class="highlight box"
  }
~~~

同样可以使用 getter 方法

~~~ts
  @HostBinding('class')  get class() {  return "highlight box"  }
~~~

其他例子

~~~ts
@HostBinding('class.highlight') get hasHighlight () { return true; } 
@HostBinding('class.box') get hasBox () { return false }
~~~

HostBinding添加的样式必须存在于宿主元素的作用域中。即highlight&box必须存在于全局样式或我们添加指令的组件中。

# 在组件中使用 HostBinding & HostListner

这些组件不过是带有模板的指令。因此，我们也可以在组件中同时使用HostBinding和HostListner。

以下是一个BoxComponent组件，它将highlight & box样式应用于宿主元素。样式highlight & box定义在组件中。
~~~ts
import { Component, HostBinding, HostListener } from '@angular/core';
 
@Component({
  selector: 'app-box',
  template: `
    <h2> This is Box Component</h2> `,
  styles: [
    `
    .highlight {
      color:green;
      display: block;
    } 
    
    .box {
      border: 1px dashed green;
    }
    `
  ],
})
export class BoxComponent {
  title = 'hostBinding';
 
  @HostBinding('class.highlight') get hasHighlight() { return true; }
  @HostBinding('class.box') get hasBox() { return true }
}
~~~

在父组件中添加如下代码
~~~html
<app-box></app-box>
~~~

运行该应用程序，您将不会看到任何边框，也不会突出显示文本。即，因为宿主元素存在于父组件（AppComponent）范围中而不存在于BoxComponent范围中。因此，BoxComponent中的任何CSS样式都不会产生任何影响

~~~css
.highlight {
  color:blue;
  display: block;
} 
 
.box {
  border: 1px solid red;
}
~~~

打卡父组件样式文件并添加以上样式，这样宿主组件可以正常显示了

