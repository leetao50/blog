---
title: 指令 模板 组件 综合应用 探索Angular DOM操作
date: 2023-03-23 11:43:41
tags:
---
文章原出处:
[Exploring Angular DOM manipulation techniques using ViewContainerRef](https://blog.angularindepth.com/exploring-angular-dom-abstractions-80b3ebcfc02)

翻译原文：[使用 ViewContainerRef 探索Angular DOM操作 · Issue #19 · giscafer/giscafer.github.io](https://github.com/giscafer/giscafer.github.io/issues/19)

Angular 版本运行在不同的平台上——在浏览器上，在移动平台上，或者在 web worker 中。因此，需要在平台特定API 和框架接口之间进行抽象级别的抽象。从 Angular 来看，这些抽象的形式有以下的参考类型: ElementRef, TemplateRef, ViewRef, ComponentRef 和 ViewContainerRef。在本文中，我们将详细介绍每个引用类型，并展示如何使用它们来操作DOM。

# @ViewChild

在探索 DOM 抽象层之前，先了解下如何在组件/指令中访问它们。Angular 提供了一种叫做 DOM querys 的技术，它们以 @ViewChild 和 @ViewChildren 装饰器（decorators）的形式出现。两者功能类似，唯一区别是 @ViewChild 返回单个引用，@ViewChildren 返回由 QueryList 对象包装好的多个引用。本文示例中主要以 ViewChild 装饰器为例，后面描述时将省略 @ 符号。

通常这些装饰器与模板引用变量（template reference variable）配合使用，模板引用变量可以理解为 DOM 元素的引用标识，类似于 html 元素的 id 属性。你可以使用模板引用（template reference）来标记一个 DOM 元素（译者注：下面示例中的#tref），并在组件/指令中使用 ViewChild 装饰器查询到它，比如：

~~~ts
@Component({
    selector: 'sample',
    template: `
        <span #tref>I am span</span>
    `
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("tref", {read: ElementRef}) tref: ElementRef;

    ngAfterViewInit(): void {
        // outputs `I am span`
        console.log(this.tref.nativeElement.textContent);
    }
}
~~~

ViewChild 装饰器基本语法是：

~~~ts
@ViewChild([reference from template], {read: [reference type]});
~~~
上例中你可以看到，我把 tref 作为模板引用名称，并将 ElementRef 与该元素联系起来。

第二个参数 read 是可选的，因为 Angular 会根据 DOM 元素的类型推断出该引用类型。例如，如果它（#tref）挂载的是类似 span 的简单 html 元素，Angular 推断为 ElementRef 类型；

如果它挂载的是 template 元素，Angular 推断为 TemplateRef 类型。

一些引用类型如 ViewContainerRef 就不可以被 Angular 推断出来，所以必须在 read 参数中显式声明。其他的如 ViewRef 不可以挂载在 DOM 元素中，所以必须手动在构造函数中编码构造出来。

现在，让我们看看应该如何获取这些引用，一起去探索吧。

# ElementRef
这是最基本的抽象类，如果你查看它的类结构，你会发现它仅仅包含与原生元素交互的方法，这对访问原生 DOM 元素很有用，比如：
~~~ts
// outputs `I am span`
console.log(this.tref.nativeElement.textContent);
~~~
然而，Angular 团队不鼓励这种写法，不仅因为这种方式会存在安全风险，而且还会让你的程序与渲染层（rendering layers）紧耦合，这样就很难实现在多平台运行相同的应用程序。我认为这个问题并不是使用 nativeElement 导致的，而是使用特定的 DOM API 造成的，例如使用了 textContent。但是后文你会看到，Angular 实现了操作 DOM 的整体思路模型，这样将不用必须调用平台指定的低层次抽象的 API，如textContent。

> 使用 ViewChild 装饰器可以返回任何 DOM 元素对应的 ElementRef，但是由于组件挂载在自定义 DOM 元素中;指令也应用在 DOM 元素上，所以组件和指令都可以通过 DI（依赖注入）获取宿主元素的 ElementRef 对象。

比如：
~~~ts
@Component({
    selector: 'sample',
    ...
export class SampleComponent{
      constructor(private       hostElement:ElementRef) {
          //outputs <sample>...</sample>
          console.log(this.hostElement.nativeElement.outerHTML);
      }
    ...
~~~
所以组件通过 DI（Dependency Injection）可以访问到它的宿主元素，但 ViewChild 装饰器经常被用来获取模板视图中的 DOM 元素。然而指令却相反，因为指令没有视图模板，所以主要用来获取挂载指令的宿主元素。

# TemplateRef

通过 template 标签，浏览器可以解析这段 html 代码，并创建对应的 DOM 树，但不会渲染它，该 DOM 树可以通过 content 属性访问：
~~~html
<script>
    let tpl = document.querySelector('#tpl');
    let container = document.querySelector('.insert-after-me');
    insertAfter(container, tpl.content);
</script>
<div class="insert-after-me"></div>
<ng-template id="tpl">
    <span>I am span in template</span>
</ng-template>
~~~
Angular 拥抱了标准，使用 TemplateRef 类来操作 template 元素，看看它是如何使用的（译者注：ng-template 是 Angular 提供的类似于 template 原生 html 标签）：

~~~ts
@Component({
    selector: 'sample',
    template: `
        <ng-template #tpl>
            <span>I am span in template</span>
        </ng-template>
    `
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("tpl") tpl: TemplateRef<any>;

    ngAfterViewInit() {
        let elementRef = this.tpl.elementRef;
        // outputs `template bindings={}`
        console.log(elementRef.nativeElement.textContent);
    }
}
~~~
Angular 框架从 DOM 中移除 template 元素，并在其位置插入注释，这是渲染后的样子：
~~~html
<sample>
    <!--template bindings={}-->
</sample>
~~~

TemplateRef 是一个结构简单的抽象类，它的 elementRef 属性是对其宿主元素的引用，它还有一个 createEmbeddedView 方法。createEmbeddedView 方法非常有用，因为它可以创建一个视图（view）并返回该视图的引用对象 ViewRef。

# ViewRef
该抽象类型表示一个 Angular 视图（View），在 Angular 世界里，视图（View）是构建应用中 UI 的基础单元。它是可以同时创建与销毁的最小元素组合。Angular 鼓励开发者把 UI 作为一堆视图（View）的组合，而不仅仅是 html 标签组成的树。

***Angular 支持两种视图类型：***

+ 内嵌视图（Embedded View），与 Template 关联
+ 宿主视图（Host View），与 Component 关联
创建内嵌视图

模板仅仅是视图的蓝图，可以通过之前提到的 createEmbeddedView 方法创建视图，比如：
~~~ts
ngAfterViewInit() {
    let view = this.tpl.createEmbeddedView(null);
}
~~~

## 创建宿主视图

宿主视图是在组件动态实例化时创建的，一个动态组件（dynamic component）可以通过 ComponentFactoryResolver 创建：
~~~ts
constructor(private injector: Injector,
            private r: ComponentFactoryResolver) {
    let factory = this.r.resolveComponentFactory(ColorComponent);
    let componentRef = factory.create(injector);
    let view = componentRef.hostView;
}
~~~
在 Angular 中，每个组件都绑定着一个指定的注入器（Injector）实例，所以创建 ColorComponent 组件时传入当前组件（即 SampleComponent）的注入器。另外，别忘了，动态创建的组件，需要在 ngModule 中或者宿主组件中增加 EntryComponents 配置。

现在，我们已经看到内嵌视图和宿主视图是如何被创建的，一旦视图被创建，它就可以使用 ViewContainer 插入 DOM 树中。下文主要探索这个功能。

# ViewContainerRef
视图容器可以挂载一个或多个视图。

首先需要说的是，任何 DOM 元素都可以作为视图容器，然而有趣的是，对于绑定 ViewContainer 的 DOM 元素，Angular 不会把视图插入该元素的内部，而是追加到该元素后面，这类似于 router-outlet 中插入组件的方式。

通常，比较好的方式是把 ViewContainer 绑定在 ng-container 元素上，因为 ng-container 元素会被渲染为注释，从而不会在 DOM 中引入多余的 html 元素。下面示例描述在组建模板中如何创建 ViewContainer：

~~~ts
@Component({
    selector: 'sample',
    template: `
        <span>I am first span</span>
        <ng-container #vc></ng-container>
        <span>I am last span</span>
    `
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("vc", {read: ViewContainerRef}) vc: ViewContainerRef;

    ngAfterViewInit(): void {
        // outputs `template bindings={}`
        console.log(this.vc.element.nativeElement.textContent);
    }
}
~~~
如同其他 DOM 抽象类一样，ViewContainer 绑定到特殊的 DOM 元素，并可以通过 element 访问到。例如上例中，它绑定到 ng-container 元素上，并且渲染为 HTML 注释，所以输出会是 template bindings={}。

## 操作视图

ViewContainer 提供了一些操作视图 API：
~~~ts
class ViewContainerRef {
    ...m
    clear() : void
    insert(viewRef: ViewRef, index?: number) : ViewRef
    get(index: number) : ViewRef
    indexOf(viewRef: ViewRef) : number
    detach(index?: number) : ViewRef
    move(viewRef: ViewRef, currentIndex: number) : ViewRef
}
~~~
从上文我们已经知道内嵌视图和宿主视图的创建方式，当创建视图后，就可以通过 insert 方法插入 DOM 中。下面示例描述如何通过 ng-template 创建内嵌视图，并在 ng-container 中插入该视图。
~~~TS
@Component({
    selector: 'sample',
    template: `
        <span>I am first span</span>
        <ng-container #vc></ng-container>
        <span>I am last span</span>
        <ng-template #tpl>
            <span>I am span in template</span>
        </ng-template>
    `
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("vc", {read: ViewContainerRef}) vc: ViewContainerRef;
    @ViewChild("tpl") tpl: TemplateRef<any>;

    ngAfterViewInit() {
        let view = this.tpl.createEmbeddedView(null);
        this.vc.insert(view);
    }
}
~~~

通过上面的实现，最后的 html 看起来是：
~~~html
<sample>
    <span>I am first span</span>
    <!--template bindings={}-->
    <span>I am span in template</span>

    <span>I am last span</span>
    <!--template bindings={}-->
</sample>
~~~
（译者注：从上文中知道是追加到 ng-container 后面，而不是插入到该 DOM 元素内部，因为在 Angular 中 ng-container 元素不会生成真实的 DOM 元素，所以在结构中不会发现这个 “追加” 的痕迹。如果把 ng-container 替换成其他元素，则可以明显地看到视图是追加在 viewContainer 之后的：
~~~html
<div _ngcontent-c4=""></div><span _ngcontent-c4>I am span in template</span>）
~~~

此外，可以通过 detach 方法从 DOM 移除视图，其他的方法可以很容易通过方法名知道其含义，如通过 index 方法获得对应索引的视图引用，move 方法移动视图位置次序，或者使用 remove 方法从移除所有的视图。

# 创建视图

ViewContainer 也提供了手动创建视图 API ：
~~~ts
class ViewContainerRef {
    element: ElementRef
    length: number

    createComponent(componentFactory...): ComponentRef<C>
    createEmbeddedView(templateRef...): EmbeddedViewRef<C>
    ...
}
~~~
上面两个方法是对上文中我们手动操作的封装，可以传入模板引用对象或组件工厂对象来创建视图，并将该视图插入视图容器中特定位置。

# ngTemplateOutlet 和 ngComponentOutlet
尽管知道 Angular 操作 DOM 的内部机制是好事，但是如果存在某种便捷的方式就更好了。Angular 提供了两种快捷指令：ngTemplateOutlet 和 ngComponentOutlet。写作本文时这两个指令都是实验性的，ngComponentOutlet 也将在Angular4.0 版本中可用。如果你读完了上文，就很容易知道这两个指令是做什么的。

## ngTemplateOutlet

该指令会把 DOM 元素标记为 ViewContainer，并插入由模板创建的内嵌视图，从而不需要在组件类中显式创建该内嵌视图。这意味着，上面实例中创建内嵌视图并插入 #vc DOM 元素的代码就可以重写为：
~~~ts
@Component({
    selector: 'sample',
    template: `
        <span>I am first span</span>
        <ng-container [ngTemplateOutlet]="tpl"></ng-container>
        <span>I am last span</span>
        <ng-template #tpl>
            <span>I am span in template</span>
        </ng-template>
    `
})
export class SampleComponent {}
~~~
从上面示例看到我们不需要在组件类中写任何实例化视图的代码，非常方便。

## ngComponentOutlet

这个指令与 ngTemplateOutlet 很相似，区别是 ngComponentOutlet 创建的是由组件实例化生成的宿主视图，不是内嵌视图。你可以这么使用：
~~~html
<ng-container *ngComponentOutlet="ColorComponent"></ng-container>
~~~
# 总结
看似有很多新知识需要消化啊，但实际上 Angular 通过视图操作 DOM 的思路模型是很清晰和连贯的。你可以使用 ViewChild 查询模板引用变量来获得 Angular DOM 元素的引用对象；DOM 元素的最简单封装是 ElementRef；而对于模板，你可以使用 TemplateRef 来创建内嵌视图；而对于组件，可以使用 ComponentRef 来创建宿主视图，同时又可以使用 ComponentFactoryResolver 创建 ComponentRef。这两个创建的视图（即内嵌视图和宿主视图）又会被 ViewContainerRef 管理。最后，Angular 又提供了两个快捷指令自动化这个过程：ngTemplateOutlet 指令使用模板创建内嵌视图；ngComponentOutlet 使用动态组件创建宿主视图。

# ViewChild与ContentChild的联系和区别
ViewChild和ContentChild其实都是从子组件中获取内容的装饰器

它们本质的区别其实就只是在于方法调用的时机以及获取内容的地方：   

1. 时机：

ViewChild在ngAfterViewInit的回调中调用

ContentChild在ngAfterContentInit的回调用调用            

2. 获取内容的地方

ViewChild从模板中获取内容

ContentChild需要从ng-content中投影的内容中获取内容，也就是没有使用ng-content投影就无法获取内容

在 ng-template 标签中，我们可以访问与外部模板中相同的上下文变量，例如变量 lesson。这是因为所有 ng-template 实例都可以访问它们所嵌入的同一个上下文。
~~~html
<ng-container *ngIf="lessons">
  <div class="lesson" *ngFor="let lesson of lessons">
    <div class="lesson-detail">
      <ng-container *ngIf="lesson else empty"> {{lesson | json}} </ng-container>
      <ng-template #empty> {{lesson | json}} </ng-template>
    </div>
  </div>
</ng-container>
~~~
但是每个模板也可以定义自己的一组输入变量! 实际上，每个模板都关联了一个上下文对象，该对象包含所有特定于模板的输入变量。
~~~ts
@Component({
  selector: "app-root",
  template: `
    <ng-template #estimateTemplate let-lessonsCounter="estimate">
      <div>Approximately {{ lessonsCounter }} lessons ...</div>
    </ng-template>
    <ng-container *ngTemplateOutlet="estimateTemplate; context: ctx">
    </ng-container>
  `,
})
export class AppComponent {
  totalEstimate = 10;
  ctx = { estimate: this.totalEstimate };
}
~~~
以下是对这个例子的分析：

与前面的模板不同，这个模板有一个输入变量（它也可以有几个）
通过 ng-template 上前缀为 let- 的属性来定义了一个输入变量 lessonsCounter
lessonsCounter 变量只能在 ng-template 内部可见
该变量的内容由赋给 let-lessonsCounter 属性的表达式 estimate 决定
estimate 表达式根据上下文对象 context 求值，并将其与要实例化的模板一起传递给 ngTemplateOutlet
上下文对象 context 有一个名为 estimate 的属性，以便在模板中显示该值
还可以给 ctx 对象添加其他属性，然后通过 ngTemplateOutlet 的 context 输入到 ng-template，从而可以拿到更多的输入变量
最终在屏幕上展示的结果是：

Approximately 10 lessons ...
这让我们对如何定义和实例化自己的模板有了一个很好的概览。使用这样的方式还可以在 component 中通过模板进行编码，接下来就会介绍。

