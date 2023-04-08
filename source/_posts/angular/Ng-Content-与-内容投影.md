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

# Contentchild & ContentChilden
ContentChild和ContentChildren是装饰器，我们使用它们来查询和获取对DOM中投影内容的引用。投影内容是组件从父组件接收的内容。

ContentChild和ContentChildren与ViewChild和ViewChildren非常相似。我们使用ViewChild或ViewChildren来查询和获取组件中DOM元素的引用。DOM元素可以是HTML元素、子组件或指令等。但是，我们不能使用ViewChild或ViewChildren来获取使用投影插入的模板实例。

# 内容投影回顾
内容投影是将HTML内容从父组件传递到子组件的一种方式。子组件将在指定位置显示投影进来的模板。我们使用ng-content元素在子组件的模板中为投影进来的模板指定一个位置。ng-content还允许我们使用选择器属性创建多个投影位置。父组件可以向每个投影位置发送不同的内容。

# ContentChild and ContentChildren 例子

为了理解ContentChild和ContentChildren是如何工作的，让我们创建一个简单的卡片应用程序。该应用程序有一个CardComponent，它显示单个卡片。

~~~ts
import { Component} from '@angular/core';
 
 
@Component({
  selector: 'card',
  template: `
 
    <div class="card">
      <ng-content select="header"></ng-content>
      <ng-content select="content"></ng-content>
      <ng-content select="footer"></ng-content>
    </div> 
   
  `,
  styles: [
    ` .card { min- width: 280px;  margin: 5px;  float:left  } 
    `
  ]
})
export class CardComponent {
 
}
~~~
该组件通过三个Ng-Content定义了多个插槽。插槽的名称分别为页眉、内容和页脚。使用组件的用户可以将内容发送到这三个插槽中的任何一个或全部。

以下代码来CardListComponent组件的，CardListComponent组件实例化了三个CardComponent组件，并分别发送了页眉、内容和页脚的内容。

此外，请注意，我们在页眉内容的h1标签上有#header模板引用变量。现在让我们在CardComponent组件中使用ContentChild来访问h1元素。

~~~ts
import { Component } from '@angular/core';
 
@Component({
  selector: 'card-list',
  template: `
  
  <h1> Card List</h1>
 
      <card>
        <header><h1 #header>Angular</h1></header>
        <content>One framework. Mobile & desktop.</content>
        <footer><b>Super-powered by Google </b></footer>
      </card>
        
      <card>
        <header><h1 #header style="color:red;">React</h1></header>
        <content>A JavaScript library for building user interfaces</content>
        <footer><b>Facebook Open Source </b></footer>
      </card>
 
      <card>
        <header> <h1 #header>Typescript</h1> </header>
        <content><a href="https://www.tektutorialshub.com/typescript-tutorial/"> Typescript</a> is a javascript for any scale</content>
        <footer><i>Microsoft </i></footer>
      </card>
 
  `,
})
export class CardListComponent {
 
}
~~~

## 使用 ContentChild 和 ContentChildren

让我们返回 CardComponent 组件

首先，导入 ContentChild 元素
~~~ts
import { Component, ContentChild, ContentChildren, ElementRef, Renderer2,  ViewChild } from '@angular/core';
~~~
然后，使用它在投影内容中查询 header元素
~~~ts
@ContentChild("header") cardContentHeader: ElementRef;
~~~
这里，cardContentHeader是变量。我们对该变量应用@ContentChild装饰器。header是我们想要读取的模板变量，它应用于h1元素上。

cardContentHeader变量无法立即使用。因为组件生命周期挂钩，angular首先初始化组件，然后它会引发ngOnChanges、ngOnInit和ngDoCheck挂钩。接下来将初始化投影的组件，然后Angular抛出AfterContentInit和AfterContentChecked钩子。因此，cardContentHeader只能在AfterContentInit挂钩之后使用。

一旦我们引用了DOM元素，我们就可以使用renderor2来操纵它的样式等。

~~~ts

ngAfterContentInit() {
   
    this.renderor.setStyle(this.cardContentHeader.nativeElement,"font-size","20px")
 
}
~~~

# ViewChild Vs ContentChild

例如，在CardComponent组件中，使用ViewChild查询来读取页眉元素。您会发现cardViewHeader是未定义
~~~ts
@ViewChild("header") cardViewHeader: ElementRef;
~~~

# ContentChild语法

ContentChild查询DOM并返回第一个匹配元素，然后更新组件中对应的变量

## 语法
ContentChild的语法如下所示。

~~~ts
ContentChild(selector: string | Function | Type<any>, opts: { read?: any; static: boolean; }): any
~~~

我们在组件属性上应用contentChild装饰器，它有两个参数，第一个参数是selector选择器，第二个参数是opts选项。

selector（查询选择器）：用于查询的指令类型或查询字符串名称

opts：有两个选项。

static：设置为True：解析查询结果在变更检测之前执行，设置为false：解析查询结果在更改检测之后执行，默认为false。

read：使用它从查询的元素中读取不同的令牌

变更检测查找第一个与selecter匹配的元素，并使用该元素更新组件中的参数。如果DOM发生更改，并且有一个新元素与selecter匹配，则变更检测会更新组件中的对应参数。

## Selector
查询选择器可以是字符串、类型或返回字符串或类型的函数。支持以下选择器。

+ 组件或指令类型
+ 作为字符串的模板引用变量

~~~ts
/Using a Template Reference Variable
@ContentChild("header") cardContentHeader: ElementRef;

//Using component/directive as type
@ContentChild(childComponent) cardChildComponent: childComponent;
~~~

## Static

确定何时解析查询。设置为True：当视图首次初始化（在第一次更改检测之前）解析，设置为False：在每次变更检测之后解析。

## Read 
使用它可以从查询的元素中读取不同的令牌。

例如，考虑以下投影内容。nameInput可以是输入元素，也可以是ngModel指令。
~~~html
<input #nameInput [(ngModel)]="name">
~~~

以下代码中的ContentChild将输入元素作为elementRef返回。
~~~ts
@ContentChild('nameInput',{static:false}) nameVar;
~~~

您可以使用read令牌来要求ContentChild返回正确的类型。
~~~ts
@ContentChild('nameInput',{static:false, read: NgModel}) nameVarAsNgModel;
@ContentChild('nameInput',{static:false, read: ElementRef}) nameVarAsElementRef;
@ContentChild('nameInput', {Static:false, read: ViewContainerRef }) nameVarAsViewContainerRef;
~~~

# ContentChildren

使用ContentChildren装饰器从投影的内容中获取元素引用的列表。

ContentChildren与ContentChild不同。ContentChild总是返回对单个元素的引用。如果存在多个元素，则ContentChild返回第一个匹配元素，ContentChildren总是将所有匹配的元素作为QueryList返回。您可以遍历列表并访问每个元素。

## 语法
contentChildren的语法如下所示。它与contentChild的语法非常相似，它没有Static选项，但又descendants选项

将descendants设为True以包括所有子元素，否则仅包括直接子元素。

ContentChildren总是在更改检测之后解析。即为什么它没有static选项。而且，您不能在ngOnInit钩子中引用它，因为它还没有初始化。

