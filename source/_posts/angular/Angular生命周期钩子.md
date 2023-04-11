---
title: Angular生命周期钩子
date: 2023-04-08 18:56:06
tags:
---
AfterViewInit、AfterContentInit、AfterViewChecked和AfterContentChecked是生命周期钩子。Angular在组件的生命周期中，调用它们。在本教程中，我们将了解它们是什么以及Angular何时调用它们。我们还了解了AfterViewInit和AfterContentInit与AfterViewChecked和AfterContentChecked之间的差异。

# 生命周期回顾
当Angular实例化组件时，组件（或指令）的生命就开始了。实例化从调用组件的构造函数开始，并通过依赖注入服务。

一旦Angular实例化了组件，它就会启动组件的变更检测，它检查并更新组件的输入数据绑定属性，并初始化组件。然后，它会引发以下生命周期钩子。

Onchanges,如果Angular检测到Input属性的任何更改或angular检测到输入变化时，Onchanges都会被运行。

OnInit，它告诉我们组件已经准备好了。这个钩子让我们有机会运行初始化逻辑，更新属性等。这个钩子只运行一次。

DoCheck，这允许我们运行自定义更改检测，因为更改检测可能会忽略一些更改。该挂钩在每个变化检测周期中运行。

在此之后，Angular又调用了四个钩子。它们是AfterContentInit、AfterContentChecked、AfterViewInit和AfterViewChecked。我们将详细研究它们。

最后，当我们移除组件时，Angular调用ngOnDestroy钩子，销毁组件。

# Content Vs View

在深入研究这些挂钩之前，我们需要了解内容(Content)和视图(View)之间的区别。钩子AfterConentInit和AfterContentChecked处理内容(Content)，而AfterViewInit、AfterViewChecked处理视图(View)。

## 内容

内容是指使用内容投影注入到该组件中的外部内容。
内容投影是将HTML内容从父组件传递到子组件的一种方式。子组件将在指定位置显示传入的内容。我们使用ng-content在子组件的模板中创建一个点，如下所示。
~~~html
<h2>Child Component</h2>
<ng-content></ng-content>   <!-- place hodler for content from parent -->
~~~

父元素在开始元素和结束元素之间注入内容。Angular将此内容传递给子组件。
~~~html
<h1>Parent Component</h1>
<app-child> This <b>content</b> is injected from parent</app-child>
 ~~~

## 视图
视图是指组件的模板。

# AfterContentInit
AfterContentInit是在组件的内容完全初始化并注入组件视图后，angular调用的生命周期挂钩。

Angular在调用AfterContentInit之前，先更新被ContentChild和ContentChildren装饰的属性。

即使组件中没有使用内容投影，Angular也会调用AfterContentInit，此挂钩在ngDoCheck挂钩之后启动。

AfterContentInit仅调用一次(在组件创建后第一个变更检测后)。

# AfterContentChecked

AfterContentChecked 在 ngDoCheck 和 AfterContentInit调用之后触发。
Angular在调用AfterContentChecked之前，先更新被ContentChild和ContentChildren装饰的属性。

# AfterViewInit
仅在组件创建后的第一个变更检测周期内触发一次。

Angular 在完成组件视图及其子视图的初始化后，在变更检测期间调用的生命周期挂钩。

在引发AfterViewInit之前，Angular 还会更新用 ViewChild 和 ViewChildren 属性装饰的属性。

# AfterViewChecked

在变更检测器完成对组件视图和子视图变更的检查后，Angular 调用的生命周期挂钩。

在引发此挂钩之前，Angular 还会更新用 ViewChild 和 ViewChildren 属性装饰的属性。

# Init Vs Checked
当第一次初始化内容或视图时，Angular 会触发 AfterContentInit 和 AfterViewInit 钩子。 这发生在第一个变化检测周期中，Angular在组件实例化后立即调用。

Angular 触发 AfterContentChecked 和 AfterViewChecked 钩子，Angular 在其中检查内容或视图是否已更改。 即先前呈现的内容或视图与当前内容或视图相同。

