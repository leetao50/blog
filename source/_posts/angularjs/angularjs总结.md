---
title: angularjs总结
date: 2024-06-04 09:52:46
tags:
---

# Html编译

在编译过程中，遇到特定的HTML结构（也就是指令）时，指令所声明的行为操作会被触发。指令可以被放在元素名，属性，类名，甚至是注释中。下面是一些等价的调用 ng-bind 指令的例子：

 ```htm
<span ng-bind="exp"></span>
<span class="ng-bind: exp;"></span>
<ng-bind></ng-bind>
<!-- directive: ng-bind exp -->
```

指令其实就是在编译器遍历DOM时碰到就需要执行的函数。

# 理解视图
绝大多数模板引擎系统采用的是把字符串模板和数据拼接，然后输出一个新的字符串，在前端这个新的字符串作为元素的 innerHTML 属性的值。

![图 0](../4b94ea5b75e38270fd8da4ff8b6a0e76eb08722e27017acfd561c359df1049f8.png)  

Angular 则不同。它的编译器直接使用DOM作为模板而不是用字符串模板。编译阶段的返回结果是一个连接函数（link func），在连接阶段会和特定的作用域中的数据模型连接生成一个实时的视图。视图和作用域数据模型的绑定是透明的。开发者不需要做任何特别的调用去更新视图。同时，我们不使用 innerHTML 属性，这样也就不会影响用户输入了。而且，Angular 指令不仅可以包含文本绑定，同时也支持行为操作的绑定

![图 1](../a43a6008749e90f7daad5d9df1cea68f21b65fd6310f1e3684dabbca062f7ff9.png)  

Angular 的这种策略生成的是稳定的DOM模板。DOM元素实例和数据模型实例的绑定在绑定期间是不会发生变化的（也就是说不是每次数据改变，最后产生的模板都要变化一次）。这就意味着在你的代码中可以去获取这些DOM模板元素并且注册相应的事件处理函数，而不用担心这个对DOM元素的引用会因为数据合并而产生变化。

# 编译指令
知道 Angular 的编译是在DOM节点上发生而非字符串上是很重要的。通常，你不会注意到这个约束，因为当一个页面加载时，浏览器自动将HTML解析为DOM树了。

然而，如果你自己手动调用 $compile 时，则需要注意上面说的注意点了。因为如果你传给它一个字符串，显然是要报错的。所以，在你传值给 $compile 之前，用 angular.element 将字符串转化为DOM。

**HTML 编译可以细分为三个阶段：**

1. $compile 遍历DOM节点，匹配指令。如果编译器发现某个元素匹配一个指令，那么这个指令就被添加到指令列表中（该列表与DOM元素对应）。一个元素可能匹配到多个指令（译注：也就是一个元素里面可能有多个指令）。

2. 当所有指令都匹配到相应的元素时，编译器按照指令的 priority 属性来排列指令的编译顺序。然后依次执行每个指令的 compile 函数。每个 compile 函数有一次更改该指令所对应的DOM模板的机会。然后，每个 compile 函数返回一个 link 函数。这些函数构成一个“合并的”连接函数，它会调用每个指令返回的 link 函数。

3. 之后，$compile 调用第二步返回的连接函数，将模板和对应的作用域连接。而这又会依次调用连接函数中包含的每个指令对应的 link 函数，进而在各个DOM元素上注册监听器以及在相应的 scope 中设置对应的 $watchs。

经过这三个阶段之后，结果是我们得到了一个作用域和DOM绑定的实时视图。所以在这之后，任一发生在已经经过编译的作用域上的数据模型的变化都会反映在DOM之中。

下面是使用 $compile 服务的相关代码。它应该能帮你理解 Angular 内部在做些什么
```javascript
  var $compile = ...; // injected into your code
  var scope = ...;
  var parent = ...; // DOM element where the compiled template can be appended
 
  var html = '<div ng-bind="exp"></div>';
 
  // Step 1: parse HTML into DOM element
  var template = angular.element(html);
 
  // Step 2: compile the template
  var linkFn = $compile(template);
 
  // Step 3: link the compiled template with the scope.
  var element = linkFn(scope);
  
  // Step 4: Append to DOM (optional)
  parent.appendChild(element);
```

