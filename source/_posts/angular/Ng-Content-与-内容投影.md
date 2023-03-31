---
title: Ng-Content 与 内容投影
date: 2023-03-30 15:59:03
tags:
---
让我们探索如何使用ng-content在模板中添加外部内容。我们知道如何使用@Input装饰器将数据从父组件传递到子组件。但它仅限于数据，我们不能使用该技术将包含HTML、CSS等元素的内容传递给子组件。要做到这一点，我们必须利用内容投影。

内容投影是将HTML内容从父组件传递到子组件的一种方式。子组件将在指定位置显示模板内容。

我们使用ng-content素在子组件的模板中指定一个位置。ng-content还允许我们使用选择器属性创建多个插入位置。

# 什么是 Ng-Content
ng-content标记充当占位符，用于插入外部或动态内容。父组件将外部内容传递给子组件。Angular解析模板时，会在子组件模板中ng-content出现的位置插入外部内容。

我们可以使用内容投影来创建一个可重用的组件。具有类似逻辑和布局的组件，可以在应用程序的许多地方使用。

以卡片组件为例。它有页眉部分、页脚部分和正文部分。这些部分的内容会有所不同。ng-content将允许我们将这些部分从父组件传递到卡片组件。这使我们能够在应用程序的许多地方使用卡片组件。

要了解内容投影是如何使用ng-content工作的，首先让我们构建一个没有ng-content的简单按钮组件。

# 无Ng-Content例子

创建一个新的angular应用程序，并创建一个新的btn.component.ts组件。这是一个简单的组件，它显示一个标题为“点击我”的按钮

~~~ts
import { Component } from '@angular/core';
 
@Component({
 selector: 'app-btn',
 template: `<button>
       Click Me
     </button>`
})
export class BtnComponent {
}
~~~

现在切换到app.component.html.

~~~html
<h2>Simple Button Demo</h2>
<app-btn></app-btn>
<app-btn></app-btn>
~~~
在上面的代码中，我们添加了两个标题为Click Me的按钮组件，他们按预期显示在屏幕上。

如果我们想从父组件更改标题，该怎么办。我们可以使用@Input属性来实现这一点。但使用@input，我们只能设置按钮的标题，无法改变子组件的外观。

# ng-content 例子
创建新的组件(FancyBtnComponent)，从上面的例子中复制全部代码，删除Click Me并添加`<ng-content></ng-content>`。此标记充当占位符。您也可以将其视为组件的一个参数。

打开  app.component.html 文件并修改内容如下：

~~~html
<h2>Button Demo With ng-content</h2>
<app-fancybtn>Click Me</app-fancybtn>
<app-fancybtn><b>Submit</b></app-fancybtn>
~~~
`<app-fancybtn></app-fancybdn>`之间的内容将传递给我们的FancyBtnComponent组件。该组件将其显示在ng-content的位置。

这种解决方案的优点是可以传递任何HTML内容。

## 事件

点击、输入等事件都会向上冒泡传递，因此可以在父对象中捕获，如下所示

~~~ts
**app.component.html**
 
<h2>Button with click event</h2>
<app-fancybtn (click)="btnClicked($event)"><b>Submit</b></app-fancybtn>
 
 
** App.component.ts ***
 
 btnClicked($event) {
   console.log($event)
   alert('button clicked')
 }
 ~~~
但是，如果您有多个按钮，那么您可以通过检查$event参数来，确定是哪个按钮触发的该事件。

## 自定义事件

你可以使用@output来创建自定义事件，如下所示

~~~TS
 
@Output() someEvent:EventEmitter =new EventEmitter();
 
raiseSomeEvent() {
 this.someEvent.emit(args);
}
~~~

在父组件中

~~~html
<app-fancybtn (someEvent)=”DoSomething($event)”><b>Submit</b></app-fancybtn>
~~~

# 使用ng-content实现多投影
如下例所示, 创建一个新的组件card.component.ts

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-card',
  template: `
    <div class="card">
    <div class="header">
      <ng-content select="header" ></ng-content>
    </div>
    <div class="content">
      <ng-content select="content" ></ng-content>
    </div>
    <div class="footer">
      <ng-content select="footer" ></ng-content>
    </div>
    </div>
  `,
  styles: [
    ` .card { min- width: 280px;  margin: 5px;  float:left  } 
      .header { color: blue}
    `
  ]
})
export class CardComponent {
}
~~~
在上面的例子中，我们有三个ng-content 标签，他们的选择器分别是 header、content、footer。

现在我们打开app.component.html文件，添加如下代码

~~~html
<app-card>
 <header><h1>Angular</h1></header>
 <content>One framework. Mobile & desktop.</content>
 <footer><b>Super-powered by Google </b></footer>
</app-card>
  
<app-card>
 <header><h1 style="color:red;">React</h1></header>
 <content>A JavaScript library for building user interfaces</content>
 <footer><b>Facebook Open Source </b></footer>
</app-card>
~~~

## Select属性是CSS选择器

您可以使用任何CSS选择器作为ng-content的select属性。比如class、element、id属性等。例如，下面使用CSS类的卡片组件。

~~~ts
import { Component } from '@angular/core';
 
@Component({
  selector: 'card',
  template: `
    <div class="card">
    <div class="header">
      <ng-content select=".header" ></ng-content>
    </div>
    <div class="content">
      <ng-content select=".content" ></ng-content>
    </div>
    <div class="footer">
      <ng-content select=".footer" ></ng-content>
    </div>
    </div>
  `,
  styles: [
    ` .card { width: 280px;  margin: 5px;  float:left; border-width:1px; border-style:solid ; } 
      .header { color: blue}
    `
  ]
})
export class CardComponent {
~~~
我们可以如下使用它。
~~~html
<card>
  <div class="header">
    <h1>Angular</h1>
  </div>
  <div class="content">One framework. Mobile & desktop.</div>
  <div class="footer"><b>Super-powered by Google </b></div>
</card>
 
<card>
  <div class="header">
    <h1 style="color:red;">React</h1>
  </div>
  <div class="content">A JavaScript library for building user interfaces</div>
  <div class="footer"><b>Facebook Open Source </b></div>
</card>
~~~

类似地，您可以使用如下所示的各种CSS选择器


~~~html
 <ng-content select="custom-element" ></ng-content>
 <ng-content select=".custom-class" ></ng-content>
 <ng-content select="[custom-attribute]" ></ng-content>
 ~~~

 ## 不带select属性的 Ng-Content 会捕获全部内嵌HTML

 在下面的示例中，最后一段HTML不属于任何ng-content，因此，ng-content不会投影最后一段，因为它无法确定添加到哪里。

 ~~~html
 <card>
  <div class="header"><h1>Typescript</h1></div>
  <div class="content">Typescript is a javascript for any scale</div>
  <div class="footer"><b>Microsoft </b></div>
  <p>This text will not be shown</p>
</card>
~~~
为了解决上述问题，我们可以添加一个没有select属性的Ng-Content。它将显示那些不能投影到其他Ng-Content中的HTML。

~~~ts
import { Component } from '@angular/core';
 
 
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
    <div class="header">
      <ng-content select="header" ></ng-content>
    </div>
    <div class="content">
      <ng-content select="content" ></ng-content>
    </div>
    <div class="footer">
      <ng-content select="footer" ></ng-content>
    </div>
    <ng-content></ng-content>                      
    </div>
  `,
  styles: [
    ` .card { min- width: 280px;  margin: 5px;  float:left  } 
      .header { color: blue}
    `
  ]
})
export class CardComponent {
}

~~~
# ngProjectAs

有时候需要使用 ng-container 包装 组件，这种情况多数是使用结构指令 如 ngif、ngSwitch等

在下面的示例中，我们将标头封装在ng-container中。

~~~html
<card>
  <ng-container>
    <div class="header">
      <h1 style="color:red;">React</h1>
    </div>
  </ng-container>
  <div class="content">A JavaScript library for building user interfaces</div>
  <div class="footer"><b>Facebook Open Source </b></div>
</card>
~~~

由于ng-container的原因，标头部分不会投影到标头标记位置。相反，它被投影到没有select选择器的ng-content中。

您可以使用ngProjectAs属性，来解决这种情况，如下所示。

~~~html
<card>
  <ng-container ngProjectAs="header">
    <div>
      <h1 style="color:red;">React</h1>
    </div>
  </ng-container>
  <div class="content">A JavaScript library for building user interfaces</div>
  <div class="footer"><b>Facebook Open Source </b></div>
</card>
~~~

# 总结
ng-content允许我们在模板中添加外部内容。与@Input不同，使用ng-content，我们可以传递包括HTML元素、CSS等的数据，这也被称为内容投影。我们还可以使用选择器属性定义不同的插入位置。这些选择器允许我们向不同的ng-content添加不同的内容。

