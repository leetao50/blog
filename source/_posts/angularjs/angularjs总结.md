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

## 理解视图
绝大多数模板引擎系统采用的是把字符串模板和数据拼接，然后输出一个新的字符串，在前端这个新的字符串作为元素的 innerHTML 属性的值。

![图 0](../4b94ea5b75e38270fd8da4ff8b6a0e76eb08722e27017acfd561c359df1049f8.png)  

Angular 则不同。它的编译器直接使用DOM作为模板而不是用字符串模板。编译阶段的返回结果是一个连接函数（link func），在连接阶段会和特定的作用域中的数据模型连接生成一个实时的视图。视图和作用域数据模型的绑定是透明的。开发者不需要做任何特别的调用去更新视图。同时，我们不使用 innerHTML 属性，这样也就不会影响用户输入了。而且，Angular 指令不仅可以包含文本绑定，同时也支持行为操作的绑定

![图 1](../a43a6008749e90f7daad5d9df1cea68f21b65fd6310f1e3684dabbca062f7ff9.png)  

Angular 的这种策略生成的是稳定的DOM模板。DOM元素实例和数据模型实例的绑定在绑定期间是不会发生变化的（也就是说不是每次数据改变，最后产生的模板都要变化一次）。这就意味着在你的代码中可以去获取这些DOM模板元素并且注册相应的事件处理函数，而不用担心这个对DOM元素的引用会因为数据合并而产生变化。

## 编译指令
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

# 控制器

在Angular中，控制器就像 JavaScript 中的构造函数一般，是用来增强 Angular作用域(scope) 的。

当一个控制器通过 ng-controller 指令被添加到DOM中时，ng 会调用该控制器的构造函数来生成一个控制器对象，这样，就创建了一个新的子级 作用域(scope)。在这个构造函数中，作用域(scope)会作为$scope参数注入其中，并允许用户代码访问它。

Angular模块 下通过 .controller 为你的应用创建控制器，如下所示：
```javascript
var myApp = angular.module('myApp',[]);
 
    myApp.controller('GreetingCtrl', ['$scope', function($scope) {
        $scope.greeting = 'Hola!';
        $scope.double = function(value) { return value * 2; };
    }]);
```
通常情况下，控制器不应被赋予太多的责任和义务，它只需要负责一个单一视图所需的业务逻辑。

**注意，下面的场合千万不要用控制器：**

+ 任何形式的DOM操作：控制器只应该包含业务逻辑。DOM操作则属于应用程序的表现层逻辑操作，向来以测试难度之高闻名于业界。把任何表现层的逻辑放到控制器中将会大大增加业务逻辑的测试难度。ng 提供数据绑定 （数据绑定） 来实现自动化的DOM操作。如果需要手动进行DOM操作，那么最好将表现层的逻辑封装在 指令 中
+ 格式化输入：使用 angular表单控件 代替
+ 过滤输出：使用 angular过滤器 代替
+ 在控制器间复用有状态或无状态的代码：使用angular服务 代替
+ 管理其它部件的生命周期（如手动创建 service 实例）

## Scope 继承
我们常常会在不同层级的DOM结构中添加控制器。由于 ng-controller 指令会创建新的子级 scope ，这样我们就会获得一个与DOM层级结构相对应的的基于继承关系的 scope 层级结构。由于 Js 是基于原型的继承，所以）底层（内层）控制器的 $scope 能够访问在高层控制器的 scope 中定义的属性和方法。
demo中的蓝色边框很清晰的展现了 scope 的层级和DOM层级的对应关系。它还展示了“scope 是由 ng-controller 指令创建并由其对应的控制器所管理”这个概念。

```htm
<!doctype html>
<html ng-app="scopeInheritance">
  <head>
    <script src="http://code.angularjs.org/1.2.25/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div ng-app="scopeInheritance" class="spicy">
      <div ng-controller="MainCtrl">
        <p>Good {{timeOfDay}}, {{name}}!</p>
    
        <div ng-controller="ChildCtrl">
          <p>Good {{timeOfDay}}, {{name}} !</p>
    
          <div ng-controller="GrandChildCtrl">
            <p>Good {{timeOfDay}}, {{name}}!</p>
          </div>
        </div>
      </div>
    </div>
  </body>
</html>
<script>
var myApp = angular.module('scopeInheritance', []);
myApp.controller('MainCtrl', ['$scope', function($scope){
  $scope.timeOfDay = 'morning';
  $scope.name = 'Nikki';
}]);
myApp.controller('ChildCtrl', ['$scope', function($scope){
  $scope.name = 'Mattie';
}]);
myApp.controller('GrandChildCtrl', ['$scope', function($scope){
  $scope.timeOfDay = 'evening';
  $scope.name = 'Gingerbreak Baby';
}]);
</script>
```
![图 2](../48ab09af690175b78dfaa2fe68faa27403d0e950a8a62b8eef57dd423bb7b002.png)  

注意，上面例子中我们在HTML模板中嵌套了三个 ng-controller 指令，这导致我们的视图中有4个 scope：

+ root scope，所有作用域的“根”
+ MainCtrl 控制器管理的 scope （简称 MainCtrl scope），拥有 timeOfDay 和 name 两个属性
ChildCtrl 控制器管理的 scope （简称 ChildCtrl scope），继承了 MainCtrl scope 中的 timeOfDay 属性，但重写了它的 name 属性
+ GrandChildCtrl 控制器管理的 scope （简称 GrandChildCtrl scope），重写了 MainCtrl scope 中的 timeOfDay 属性和 ChildCtrl scope 中的 name 属性

# 服务

# 作用域

作用域(Scope)是什么
+ 是一个存储应用数据模型的对象
+ 为 表达式 提供了一个执行上下文
+ 作用域的层级结构对应于 DOM 树结构
+ 作用域可以监听 表达式 的变化并传播事件

作用域有什么
+ 作用域提供了 ($watch) 方法监听数据模型的变化
+ 作用域提供了 ($apply) 方法把不是由Angular触发的数据模型的改变引入Angular的控制范围内（如控制器，服务，及Angular事件处理器等）
+ 作用域提供了基于原型链继承其父作用域属性的机制，就算是嵌套于独立的应用组件中的作用域也可以访问共享的数据模型（这个涉及到指令间嵌套时作用域的几种模式）
+ 作用域提供了 表达式 的执行环境，比如像 {{username}} 这个表达式，必须得是在一个拥有属性这个属性的作用域中执行才会有意义，也就是说，作用域中可能会像这样 scope.username 或是 $scope.username，至于有没有 $ 符号，看你是在哪里访问作用域了

控制器与指令是相互分离的，而且它们与视图之间也是分离的，这样的分离，或者说耦合度低，可以大大提高对应用进行测试的工作效率。

其实可以很简单地理解为有以下两个链条关系：

控制器 --> 作用域 --> 视图（DOM）
指令 --> 作用域 --> 视图（DOM）

# 表单

# 模块
大多数应用程序都有个 main 函数来初始化、连接以及启动整个应用。ng 中虽然没有 main 函数，但它用模块来描述应用将如何启动。这种策略有如下几种优势：

+ 整个过程是声明式的，更容易理解
+ 在单元测试中，没有必要加载所有模块，这样有利于单元测试的代码书写
+ 在场景测试中，额外的模块可以被加载进来，进而重写一些配置，这样有助于实现应用的端到端的测试
+ 第三方代码可以很容易被打包成可重用的模块
+ 模块可以用任意顺序或并行顺序加载（得益于模块执行的延迟性）

## 模块加载及依赖
模块是配置代码块和运行代码块的集合，在启动阶段被执行。最简单的模块中包含下面两种代码块：

+ 配置代码块 - 在 provider 注册和配置阶段执行（注：provider 是 ng 服务的一种）。只有 provider 和 constant 可以被注入配置代码块。这是为了防止服务在完全配置好之前被意外地初始化。
+ 执行代码块 - 在 injector 被创建后执行，被用来启动整个应用。只有服务的实例对象以及 constant 可以被注入到执行代码块。这是为了防止在应用执行期间系统的更进一步的配置。

## 模块依赖

模块声明时可以列出它所需要依赖的其它模块。一个模块依赖某模块，意味着这个被依赖的模块需要在模块被加载之前加载完毕。更具体些，假设模块A依赖于模块B，那么模块A的配置代码块的执行，必须发生在模块B的配置代码块之后；模块A的执行代码块亦同理，也在模块B的执行代码块之后被执行。每个模块只能被加载一次，即使有多个别的模块依赖它。

## 创建模块 vs 获取模块

注意，使用 angular.module('myModule', []) 将创建名为 myModule 的模块并重写已有的同名模块。而使用 angular.module('myModule') 则只会获取已有的模块实例。

  ```javascript
var myModule = angular.module('myModule', []);
  
  // 添加一些指令和服务
  myModule.service('myService', ...);
  myModule.directive('myDirective', ...);
 
  // 创建一个新模块将覆盖掉这些指令和服务
  var myModule = angular.module('myModule', []);
 
  // 由于myOtherModule模块还没有定义，所以会抛出一个异常
  var myModule = angular.module('myOtherModule');
```
# 表达式

"表达式"是一种类似JavaScript的代码片段，通常在视图中以''的形式使用。表达式由$parse服务解析，而解析之后经常会使用过滤器来格式化成一种更加用户友好的形式。

## Angular表达式与JS表达式

Angular表达式与JavaScript表达式有如下的区别：

+ 属性解析： 所有的属性的解析都是相对于作用域(scope)的，而不像JavaScript中的表达式解析那样是相对于全局'window'对象的。

+ 容错性： 表达式的解析对'undefined'和'null'具有容错性，这不像在JavaScript中，试图解析未定义的属性时会抛出ReferenceError或TypeError错误.

+ 禁止控制流语句： 表达式中不允许包括下列语句：条件判断(if)，循环(for/while)，抛出异常(throw)。

另一方面，如果你想执行特定的JavaScript代码，你应该在一个控制器里导出一个方法，然后在模板中调用这个方法。如果你想在JavaScript中解析一个Angular表达式，使用$eval()方法。

## 属性解析
属性解析发生在scope上。不像在JavaScript中，将默认的属性解析放在全局的window对象中，Angular表达式必须使用$window来指向全局的'window'对象。例如，如果你想调用在'window'上定义的'alert()'方法，在表达式中，你必须使用'$window.alert()'。作者故意这样设定，就是为了防止对全局状态非正常的访问。

```javascript
function Cntl1($window, $scope){
  $scope.name = 'World';
 
  $scope.greet = function() {
    ($window.mockWindow || $window).alert('Hello ' + $scope.name);
  }
}
```

# 供应者
在基于Angular框架的应用里，对象大都是通过注入服务自动地实例化并绑定在一起。

注入器创建两类对象，服务和专用对象。

服务是对象，而这些对象的API是由编写服务的开发人员所决定的。

专用对象遵循Angular框架特定的API。这些对象包括控制器，指令，过滤器或动画。

注入器需要知道如何去创建这些对象。你应该通过注册一种“图纸”来告诉Angular如何创建你的对象。这里共有5种图纸。

最冗长同时又最复杂的图纸是Provider图纸，其余4种分别是 —— Value，Factory，Service和Constant，这4种都只是基于Provider之上的语法糖。

总结如下最重要的几点：

+ 注入器使用图纸创建两类对象：服务和专用对象。
+ 总共有5类图纸来定义如何创建对象：Value，Factory，Service，Provicer以及Constant。
+ Factory和Service是最常用的图纸。它们之间的唯一区别就是Service图纸在创建自定义对象时更适用，而Factory还可以创建Javascript原始类型以及函数。
+ Provider图纸是最核心的图纸类型，而其它所有图纸都只是基于它的语法糖。
+ Provider也是最复杂的图纸类型，除非你正在构建需要全局配置的可复用代码，否则不要使用它。
+ 除了控制器，其他所有专用对象都是通过Factory图纸来定义的。

|特性 / 图纸类型|	Factory|	Service|	Value|	Constant|	Provider|
|-|-|-|-|-|-|
|可以有依赖|	是|	是|	否|	否|	是|
|使用依赖注入友好|	否|	是|	是*|	是*|	否|
|对象在配置阶段可访问|	否|	否|	否|	是|	是**|
|可以创建函数/原始类型|	是|	否|	是|	是|	是|

* 有直接使用new操作符预先初始化的开销。

** 在配置阶段，服务对象是不能被访问的，但Provider实例是可以被访问的。（参见我们上面列举的unicornLauncherProvider例子）。

# $location

$location服务负责解析浏览器地址栏中的URL（基于window.location），以便你的应用可以访问它。 这是一个双向同步机制 —— 对地址栏URL的任何修改都会被映射到$location服务中，对$location的任何修改也同样会被映射到地址栏。

## 我该什么时候使用$location？
只要你的应用想更改当前地址，或者你想要修改浏览器的当前地址，就用它吧！

## 什么时候不该用它？
在浏览器地址变化的时候，它不会导致全页面刷新。要想在地址变化之后重载整个页面，请使用底层API$window.location.href。


