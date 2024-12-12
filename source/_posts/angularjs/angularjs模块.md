---
title: angularjs模块
date: 2024-08-08 18:14:33
tags:
---
# 模块

在AngularJS中，模块是定义应用的最主要方式。模块包含了主要的应用代码。一个应用可以包含多个模块，每一个模块都包含了定义具体功能的代码。使用模块能给我们带来许多好处，比如：
+ 保持全局命名空间的清洁；
+ 编写测试代码更容易，并能保持其清洁，以便更容易找到互相隔离的功能；
+ 易于在不同应用间复用代码；
+ 使应用能够以任意顺序加载代码的各个部分。


## 创建模块

下面是angular.module(name:string,requires:string[])的参数列表。

**参数**

name（字符串）：name是模块的名称，字符串变量。
requires（字符串数组）：requires包含了一个字符串变量组成的列表，每个元素都是一个模块名称，本模块依赖于这
些模块，依赖需要在本模块加载之前由注入器进行预加载。

AngularJS允许我们使用angular.module()方法来声明模块，这个方法能够接受两个参数，第一个是模块的名称，第二个是依赖列表，也就是可以被注入到模块中的对象列表。

```javascript
angular.module('myApp', []);
```
这个方法相当于AngularJS模块的setter方法，是用来定义模块的。

## 读取模块

调用这个方法时如果只传递一个参数，就可以用它来引用模块。例如，可以通过以下代码来引用myApp模块：
```javascript
// 这个方法用于获取应用
angular.module('myApp')
```

这个方法相当于AngularJS模块的getter方法，用来获取对模块的引用。

# 作用域
作用域是视图和控制器之间的胶水。在应用将视图渲染并呈献给用户之前，视图中的模板会和作用域进行连接，然后应用会对DOM进行设置以便将属性变化通知给AngularJS。

作用域是应用状态的基础。基于动态绑定，我们可以依赖视图在修改数据时立刻更新`$scope`，也可以依赖`$scope`在其发生变化时立刻重新渲染视图。

AngularJS将`$scope`设计成和DOM类似的结构，因此`$scope`可以进行嵌套，也就是说我们可以引用父级`$scope`中的属性。

将应用的业务逻辑都放在控制器中，而将相关的数据都放在控制器的作用域中，这是非常完美的架构。

AngularJS启动并生成视图时，会将根ng-app元素同`$rootScope`进行绑定。`$rootScope`是所有`$scope`对象的最上层。`$rootScope`是AngularJS中最接近全局作用域的对象。在`$rootScope`上附加太多业务逻并不是好主意，这与污染JavaScript的全局作用域是一样的。

`$scope`对象在AngularJS中充当数据模型，但与传统的数据模型不一样，`$scope`并不负责处理和操作数据，它只是视图和HTML之间的桥梁，它是视图和控制器之间的胶水。`$scope`的所有属性，都可以自动被视图访问到。假设我们有如下的HTML：

```htm
<div ng-app="myApp">
 <h1>Hello {{ name }}</h1>
</div>

<!--我们希望{{ name }}变量是本地$scope的一个属性。-->
<script>
angular.module('myApp', [])
 .run(function($rootScope) {
 $rootScope.name = "World";
}); 
```

## 作用域能做什么
作用域有以下的基本功能：
+ 提供观察者以监视数据模型的变化；
+ 可以将数据模型的变化通知给整个应用，甚至是系统外的组件；
+ 可以进行嵌套，隔离业务功能和数据；
+ 给表达式提供运算时所需的执行环境。

# 控制器

控制器在AngularJS中的作用是增强视图。AngularJS中的控制器是一个函数，用来向视图的作用域中添加额外的功能。我们用它来给作用域对象设置初始状态，并添加自定义行为。当我们在页面上创建一个新的控制器时，AngularJS会生成并传递一个新的`$scope`给这个控制器。可以在这个控制器里初始化`$scope`。由于AngularJS会负责处理控制器的实例化过程，我们只需编写构造函数即可。

AngularJS同其他JavaScript框架最主要的一个区别就是，控制器并不适合用来执行DOM操作、格式化或数据操作，以及除存储数据模型之外的状态维护操作。它只是视图和`$scope`之间的桥梁。

AngularJS允许在`$scope`上设置包括对象在内的任何类型的数据，并且在视图中还可以展示对象的属性。

例如，我们在MyController上创建一个person对象，这个对象只有name这一个属性：
```javascript
app.controller('MyController', function($scope) {
 $scope.person = {
 name: 'Ari Lerner'
 };
});
```
在拥有ng-controller='MyController'这个属性的元素内部的任何子元素中，都可以访问
person对象，因为它是定义在`$scope`上的。
例如，可以方便地在视图中引用person或person.name。
```htm
<div ng-app="myApp">
 <div ng-controller="MyController">
 <h1>{{ person }}</h1>
 and their name:
 <h2>{{ person.name }}</h2>
 </div>
</div>
```
## 控制器嵌套（作用域包含作用域）

所有的作用域都通过原型继承而来，也就是说它们都可以访问父级作用域。

默认情况下，AngularJS在当前作用域中无法找到某个属性时，便会在父级作用域中进行查找。如果AngularJS找不到对应的属性，会顺着父级作用域一直向上寻找，直到抵达`$rootScope`为止。如果在`$rootScope`中也找不到，程序会继续运行，但视图无法更新。

# 指令
## 自定义指令
基于我们对HTML元素的理解，指令本质上就是AngularJS扩展具有自定义功能的HTML元素的途径。

myDirective指令的定义看起来是这样的：
```javascript
angular.module('myApp',[])
    .directive('myDirective', function() {
        return {
            restrict: 'E',
            template: '<a href="http://google.com">Click me to go to Google</a>'
        };
});
```
通过AngularJS模块API中的.directive()方法，我们可以通过传入一个字符串和一个函数来注册一个新指令。其中字符串是这个指令的名字，指令名应该是驼峰命名风格的，函数应该返回一个对象。

+ replace：方法会用自定义元素取代指令声明，而不是嵌套在其内部

声明指令本质上是在HTML中通过元素、属性、类或注释来添加功能。

下面都是用来声明前面创建指令的合法格式：
```htm
<my-directive></my-directive>
<div my-directive></div>
<div class="my-directive"></div>
<!--directive:my-directive-->
```
为了让AngularJS能够调用我们的指令，需要修改指令定义中的restrict设置。这个设置告诉AngularJS在编译HTML时用哪种声明格式来匹配指令定义。我们可以指定一个或多个格式。例如，之前创建的指令中可以指定以元素（E）、属性（A）、类（C）或注释（M）的格式来调用指令：

```javascript
angular.module('myApp', [])
    .directive('myDirective', function() {
        return {
            restrict: 'EAC',
            replace: true,
            template: '<a href="http://google.com">Click me to go to Google</a>'
        };
    });
```
无论有多少种方式可以声明指令，我们坚持使用属性方式，因为它有比较好的跨浏览器兼容性：

```htm
<div my-directive></div>
```
为了更加明确我们的意图，将restrict设置为字母A（代表attribute）：restrict: 'A' .

默认情况下约定DOM属性和JavaScript中对象属性的名字是一样的（除非对象的属性名采用的是驼峰式写法）。由于作用域中属性经常是私有的，因此可以（虽然不常见）指定我们希望将这个内部属性同哪个DOM属性进行绑定：
```javascript
scope: {
 myUrl: '@someAttr',
 myLinkText: '@'
}
```
上面的隔离作用域中的内容是：将指令的私有属性$scope.myUrl同DOM中some-attr属性的值绑定起来。这个值既可以是硬编码的也可以是当前作用域（例如Some-attr="{{expression}}）中某个表达式的运算结果。在DOM中要用some-attr代替my-url：
```htm
<div my-directive
 some-attr="http://google.com"
 my-link-text="Click me to go to Google" >
</div>
```
更进一步，还可以在DOM对应的作用域上运算表达式，并将结果传递给指令，在指令内部最终被绑定在属性上：
```htm
<div my-directive
 some-attr="{{ 'http://' + 'google.com' }}">
</div> 
```

通过这两个输入字段可以方便地观察作用域是如何在DOM中通过原型继承链接在一起的：
```html
<label>Their URL field:</label>
<input type="text" ng-model="theirUrl">
<div my-directive
 some-attr="theirUrl"
 my-link-text="Click me to go to Google"></div>

<script>
angular.module('myApp', [])
    .directive('myDirective', function() {
        return {
            restrict: 'A',
            replace: true,
            scope: {
                myUrl: '=someAttr', // 经过了修改
                myLinkText: '@'
            },
            template: '\
                <div>\
                <label>My Url Field:</label>\
                <input type="text"\
                ng-model="myUrl" />\
                <a href="{{myUrl}}">{{myLinkText}}</a>\
                </div>\
        };
    });
</script>
```

# 内置指令
注意，所有以ng前缀开头作为命名空间的指令都是AngularJS提供的内置指令，因此不要把你自己开发的指令以这个前缀命名。

基础 ng 属性指令
+ ng-href；
+ ng-src；
+ ng-disabled；
+ ng-checked；
+ ng-readonly；
+ ng-selected；
+ ng-class；
+ ng-style。

## 布尔属性
根据HTML标准的定义，布尔属性代表一个true或false值。当这个属性出现时，这个属性的值就是true（无论实际定义的值是什么）。如果未出现，这个属性的值就是false。

当在AngularJS中使用动态数据绑定时，不能简单地将这个属性值设置为ture或false，因为根据标准定义只有当这个属性不出现时，它的值才为false。因此AngularJS提供了一组带有ng前缀版本的布尔属性，通过运算表达式的值来决定在目标元素上是插入还是移除对应的属性。

1. ng-disabled
使用ng-disabled可以把disabled属性绑定到以下表单输入字段上：
```htm
<input> （text、checkbox、radio、number、url, email、submit）；
<textarea>；
<select>；
<button>。
```

通过ng-disabled可以对是否出现属性进行绑定。例如，在下面的例子中按钮会一直禁用，直到用户在文本字段中输入内容：

```htm
<input type="text" ng-model="someProperty" placeholder="TypetoEnable">
<button ng-model="button" ng-disabled="!someProperty">AButton</button>
```
2. ng-readonly
同其他布尔属性一样，HTML定义只会检查readonly属性是否出现，而不是它的实际值。通过ng-readonly可以将某个返回真或假的表达式同是否出现readonly属性进行绑定：

```htm
Type here to make sibling readonly:
<input type="text" ng-model="someProperty"><br/>
<input type="text"
 ng-readonly="someProperty"
 value="Some text here"/>
```
3. ng-checked
标准的checked属性是一个布尔属性，不需要进行赋值。通过ng-checked将某个表达式同是否出现checked属性进行绑定。

在下面的例子中，我们通过ng-init指令将someProperty的值设置为true。将some Property 同ng-checked绑定在一起，AngularJS会输出标准的HTML checked属性，这样默认会把复选框勾选：
```htm
<label>someProperty = {{someProperty}}</label>
<input type="checkbox"
 ng-checked="someProperty"
 ng-init="someProperty = true"
 ng-model="someProperty">
```
下面的例子刚好相反：
```htm
<label>someProperty = {{anotherProperty}}</label>
<input type="checkbox"
 ng-checked="anotherProperty"
 ng-init="anotherProperty = false"
 ng-model="anotherProperty">
```
注意，为了展示，这里用ng-model把someProperty和anotherProperty的值绑定到了对应的
`<label>`标签里。

4. ng-selected
ng-selected可以对是否出现option标签的selected属性进行绑定：
```htm
<label>Select Two Fish:</label>
<input type="checkbox"
 ng-model="isTwoFish">
<br/>
<select>
 <option>One Fish</option>
 <option ng-selected="isTwoFish">Two Fish</option>
</select>
```

## 类布尔属性
ng-href、ng-src等属性虽然不是标准的HTML布尔属性，但是由于行为相似，所以在AngularJS源码内部是和布尔属性同等对待的，下面介绍这些属性。ng-href和ng-src都能有效帮助重构和避免错误，因此在改进代码时强烈建议用它们代替原
来的href和src属性。

1. ng-href
当使用当前作用域中的属性动态创建URL时，应该用ng-href代替href。

2. ng-src
AngularJS会告诉浏览器在ng-src对应的表达式生效之前不要加载图像：

## 在指令中使用子作用域

1. ng-app
任何具有ng-app属性的DOM元素将被标记为$rootScope的起始点。

$rootScope是作用域链的起始点，任何嵌套在ng-app内的指令都会继承它。

2. ng-controller
内置指令ng-controller的作用是为嵌套在其中的指令创建一个子作用域，避免将所有操作和模型都定义在`$rootScope`上。用这个指令可以在一个DOM元素上放置控制器。ng-controller接受一个参数expression，这个参数是必需的。expression参数是一个AngularJS表达式。

子`$scope`只是一个JavaScript对象，其中含有从父级`$scope`中通过原型继承得到的方法和属性，包括应用的`$rootScope`。

嵌套在ng-controller中的指令有访问新子`$scope`的权限，但是要牢记每个指令都应该遵守的和作用域相关的规则。

这个例子中，在已有的控制器中嵌套了第二个控制器，并且没有设置模型对象的属性：

```htm
<div ng-controller="SomeController">
    {{ someBareValue }}
    <button ng-click="someAction()">Communicate to child</button>
    <div ng-controller="ChildController">
        {{ someBareValue }}
        <button ng-click="childAction()">Communicate to parent</button>
    </div>
</div>
<script>
    angular.module('myApp', [])
        .controller('SomeController', function($scope) {
            // 反模式，裸值
            $scope.someBareValue = 'hello computer';
            // 设置 $scope 本身的操作，这样没问题
            $scope.someAction = function() {
            // 在SomeController和ChildController中设置{{ someBareValue }}
                $scope.someBareValue = 'hello human, from parent';
            };
        })
        .controller('ChildController', function($scope) {
            $scope.childAction = function() {
            // 在ChildController中设置{{ someBareValue }}
            $scope.someBareValue = 'hello human, from child';
        };
    });
</script>
```

由于原型继承的关系，修改父级对象中的someBareValue会同时修改子对象中的值，但反之
则不行。

可以看下这个例子的实际效果，首先点击child button，然后点击parent button。这个例子充分
说明了子控制器是复制而非引用someBareValue。

如果将模型对象的某个属性设置为字符串，它会通过引用进行共享，因此在子$scope中修改属性也会修改父$scope中的这个属性。下面的例子展示了正确的做法：
```htm

<div ng-controller="SomeController">
    {{ someModel.someValue }}
    <button ng-click="someAction()">Communicate to child</button>
        <div ng-controller="ChildController">
        {{ someModel.someValue }}
        <button ng-click="childAction()">Communicate to parent</button>
    </div>
</div>
<script>
    angular.module('myApp', [])
        .controller('SomeController', function($scope) {
            // 最佳实践，永远使用一个模式
            $scope.someModel = {
                someValue: 'hello computer'
            }
            $scope.someAction = function() {
                $scope.someModel.someValue = 'hello human, from parent';
            };
        })
        .controller('ChildController', function($scope) {
            $scope.childAction = function() {
            $scope.someModel.someValue = 'hello human, from child';
        };
    }); 
</script>
```

无论点击哪个按钮，值都会进行同步修改。

虽然这个特性是使用ng-controller时最重要的特性之一，但在使用任何会创建子作用域的指令时，如果将指令定义中的scope设置为true，这个特性也会带来负面影响。

3. ng-include
使用ng-include可以加载、编译并包含外部HTML片段到当前的应用中。模板的URL被限制在与应用文档相同的域和协议下，可以通过白名单或包装成被信任的值来突破限制。

要记住，使用ng-include时AngularJS会自动创建一个子作用域。如果你想使用某个特定的作用域，例如ControllerA的作用域，必须在同一个DOM元素上添加ng-controller ="ControllerA"指令，这样当模板加载完成后，不会像往常一样从外部作用域继承并创建一个新的子作用域。下面看一个例子：
```htm
<div ng-include="/myTemplateName.html"
 ng-controller="MyController"
 ng-init="name = 'World'">
 Hello {{ name }}
</div>
```

4. ng-switch
这个指令和ng-switch-when及on="propertyName"一起使用，可以在propertyName发生变化时渲染不同指令到视图中。在下面的例子中，当person.name是Ari时，文本域下面的div会显示出来，并且这个人会获得胜利：
```htm

<input type="text" ng-model="person.name"/>
<div ng-switch on="person.name">
    <p ng-switch-default>And the winner is</p>
    <h1 ng-switch-when="Ari">{{ person.name }}</h1>
</div>
```

注意，在switch被调用之前我们用ng-switch-default来输出默认值。

5. ng-view
ng-view指令用来设置将被路由管理和放置在HTML中的视图的位置。

6. ng-if
使用ng-if指令可以完全根据表达式的值在DOM中生成或移除一个元素。如果赋值给ng-if的表达式的值是false，那对应的元素将会从DOM中移除，否则对应元素的一个克隆将被重新插入DOM中。

ng-if同no-show和ng-hide指令最本质的区别是，它不是通过CSS显示或隐藏DOM节点，而是真正生成或移除节点。当一个元素被ng-if从DOM中移除，同它关联的作用域也会被销毁。而且当它重新加入DOM中时，会通过原型继承从它的父作用域生成一个新的作用域。

同时有一个重要的细节需要知道，ngIf重新创建元素时用的是它们编译后的状态。如果ng-if内部的代码加载之后被jQuery修改过（例如用.addClass），那么当ng-if的表达式值为false时，这个DOM元素会被移除，表达式再次成为true时这个元素及其内部的子元素会被重新插入DOM，此时这些元素的状态会是它们的原始状态，而不是它们上次被移除时的状态。也就是说
无论用jQuery的.addClass添加了什么类都不会存在了。

7. ng-repeat
ng-repeat用来遍历一个集合或为集合中的每个元素生成一个模板实例。集合中的每个元素都会被赋予自己的模板和作用域。同时每个模板实例的作用域中都会暴露一些特殊的属性。
+ `$index`：遍历的进度（0...length-1）。
+ `$first`：当元素是遍历的第一个时值为true。
+ `$middle`：当元素处于第一个和最后元素之间时值为true。
+ `$last`：当元素是遍历的最后一个时值为true。
+ `$even`：当`$index`值是偶数时值为true。
+ `$odd`：当`$index`值是奇数时值为true。
  
下面的例子展示了如何用`$odd`和`$even`来制作一个红蓝相间的列表。记住，JavaScript中数组的索引从0开始，因此我们用`!$even`和`!$odd`来将`$even`和`$odd`的布尔值反转。
```htm
<ul ng-controller="PeopleController">
 <li ng-repeat="person in people" ng-class="{even: !$even, odd: !$odd}">
 {{person.name}} lives in {{person.city}}
 </li>
</ul>
<script>
    .odd {
        background-color: blue;
    }
    .even {
        background-color: red;
    }
    angular.module('myApp',[])
        .controller('PeopleController',function($scope) {
        $scope.people = [
            {name: "Ari", city: "San Francisco"},
            {name: "Erik", city: "Seattle"}
        ];
    }); 
</script>
```
8. ng-init
ng-init指令用来在指令被调用时设置内部作用域的初始状态

```htm
<div ng-init="greeting='Hello';person='World'">
 {{greeting}} {{person}}
</div> 
```

9. `{{ }}`
```htm
<div>{{name}}</div>
```
`{{ }}`语法是AngularJS内置的模板语法，它会在内部`$scope`和视图之间创建绑定。基于这个绑定，只要`$scope`发生变化，视图就会随之自动更新。事实上它也是指令，虽然看起来并不像，实际上它是ng-bind的简略形式，用这种形式不需要创建新的元素，因此它常被用在行内文本中。

注意，在屏幕可视的区域内使用`{{ }}`会导致页面加载时未渲染的元素发生闪烁，用ng-bind可以避免这个问题。
```htm
<body ng-init="greeting='HelloWorld'">
 {{ greeting }}
</body> 
```
10. ng-bind
尽管可以在视图中使用`{{ }}`模板语法（AngularJS内置的方式），我们也可以通过ng-bind指令实现同样的行为。
```htm
<body ng-init="greeting='HelloWorld'">
 <p ng-bind="greeting"></p>
</body> 
```
11. ng-cloak
除使用ng-bind来避免未渲染元素闪烁，还可以在含有{{ }}的元素上使用ng-cloak指令：
```htm
<body ng-init="greeting='HelloWorld'">
 <p ng-cloak>{{ greeting }}</p>
</body>
```
ng-cloak指令会将内部元素隐藏，直到路由调用对应的页面时才显示出来。

12. ng-bind-template
同ng-bind指令类似，ng-bind-template用来在视图中绑定多个表达式。
```htm
<div ng-bind-template="{{message}}{{name}}">
</div>
```
13. ng-model
ng-model指令用来将input、select、text area或自定义表单控件同包含它们的作用域中的属性进行绑定。它可以提供并处理表单验证功能，在元素上设置相关的CSS类（ng-valid、ng-invalid等），并负责在父表单中注册控件。

它将当前作用域中运算表达式的值同给定的元素进行绑定。如果属性并不存在，它会隐式创建并将其添加到当前作用域中。

我们应该始终用ngModel来绑定`$scope`上一个数据模型内的属性，而不是`$scope`上的属性，这可以避免在作用域或后代作用域中发生属性覆盖。

14. ng-show/ng-hide
ng-show和ng-hide根据所给表达式的值来显示或隐藏HTML元素。当赋值给ng-show指令的值为false时元素会被隐藏。类似地，当赋值给ng-hide指令的值为true时元素也会被隐藏。

元素的显示或隐藏是通过移除或添加ng-hide这个CSS类来实现的。.ng-hide类被预先定义在了AngularJS的CSS文件中，并且它的display属性的值为none（用了!important标记）。

15. ng-change
这个指令会在表单输入发生变化时计算给定表达式的值。因为要处理表单输入，这个指令要和ngModel联合起来使用。
```htm
<div ng-controller="EquationController">
 <input type="text" ng-model="equation.x"
 ng-change="change()" />
 <code>{{ equation.output }}</code>
</div>
<script>
angular.module('myApp',[])
    .controller('EquationController',function($scope) {
        $scope.equation = {};
        $scope.change = function() {
            $scope.equation.output = parseInt($scope.equation.x) + 2;
        };
    });
</script>
```

上面的例子中，只要文本输入字段中的内容发生了变化就会改变equation.x的值，进而运行change()函数。

16. ng-form
ng-form用来在一个表单内部嵌套另一个表单。普通的HTML <form>标签不允许嵌套，但ng-form可以。这意味着内部所有的子表单都合法时，外部的表单才会合法。这对于用ng-repeat动态创建表单是非常有用的。

17. ng-click
ng-click用来指定一个元素被点击时调用的方法或表达式。

18. ng-select
ng-select用来将数据同HTML的<select>元素进行绑定。这个指令可以和ng-model以及ng-options指令一同使用，构建精细且表现优良的动态表单。

19. ng-submit
ng-submit用来将表达式同onsubmit事件进行绑定。这个指令同时会阻止默认行为（发送请求并重新加载页面），除非表单不含有action属性。

20. ng-class
使用ng-class 动态设置元素的类，方法是绑定一个代表所有需要添加的类的表达式。重复的类不会添加。当表达式发生变化，先前添加的类会被移除，新类会被添加。

21. ng-attr-(suffix)
当AngularJS编译DOM时会查找花括号{{ some expression }}内的表达式。这些表达式会被自动注册到`$watch`服务中并更新到`$digest`循环中，成为它的一部分

# 指令详解
## 指令定义
对于指令，可以把它简单的理解成在特定DOM元素上运行的函数，指令可以扩展这个元素的功能。

例如，ng-click可以让一个元素能够监听click事件，并在接收到事件的时候执行AngularJS表达式。正是指令使得AngularJS这个框架变得强大，并且正如所见，我们可以自己创造新的指令。

directive()这个方法是用来定义指令的：

```javascript
angular.module('myApp', [])
    .directive('myDirective', function ($timeout, UserDefinedService) {
        // 指令定义放在这里
        // 一个指令定义对象
        return {
            // 通过设置项来定义指令，在这里进行覆写
        };
    });
```
directive() 方法可以接受两个参数：
1. name（字符串）：指令的名字，用来在视图中引用特定的指令。
2. factory_function （函数）：这个函数返回一个对象，其中定义了指令的全部行为。`$compile`服务利用这个方法返回的对象，在DOM调用指令时来构造指令的行为。

当AngularJS启动应用时，它会把第一个参数当作一个字符串，并以此字符串为名来注册第二个参数返回的对象。AngularJS编译器会解析主HTML的DOM中的元素、属性、注释和CSS类名中使用了这个名字的地方，并在这些地方引用对应的指令。当它找到某个已知的指令时，就会在页面中插入指令所对应的DOM元素。

指令的工厂函数只会在编译器第一次匹配到这个指令时调用一次。和controller函数类似，我们通过$injetor.invoke来调用指令的工厂函数。

当AngularJS在DOM中遇到具名的指令时，会去匹配已经注册过的指令，并通过名字在注册过的对象中查找。此时，就开始了一个指令的生命周期，指令的生命周期开始于$compile方法并结束于link方法。

## 定义一个指令时可以使用的全部设置选项

可能的选项如下所示，每个键的值说明了可以将这个属性设置为何种类型或者什么样的函数：

```javascript
angular.module('myApp', [])
    .directive('myDirective', function() {
        return {
            restrict: String,
            priority: Number,
            terminal: Boolean,
            template: String or Template Function:function(tElement, tAttrs) (...},
            templateUrl: String,
            replace: Boolean or String,
            scope: Boolean or Object,
            transclude: Boolean,
            controller: String or function(scope, element, attrs, transclude,otherInjectables) { ... },
            controllerAs: String,
            require: String,
            link: function(scope, iElement, iAttrs) { ... },
            compile: // 返回一个对象或连接函数，如下所示：
                function(tElement, tAttrs, transclude) {
                    return {
                        pre: function(scope, iElement, iAttrs, controller) { ... },
                        post: function(scope, iElement, iAttrs, controller) { ... }
                        }
                    // 或者
                    return function postLink(...) { ... }
                }
    });
```

+ restrict：是一个可选的参数。它告诉AngularJS这个指令在DOM中可以何种形式被声明。默认AngularJS认为restrict的值是A，即以属性的形式来进行声明。

+ priority优先级（数值型）：优先级参数可以被设置为一个数值。大多数指令会忽略这个参数，使用默认值0。如果一个元素上具有两个优先级相同的指令，声明在前面的那个会被优先调用。其他情况下，具有更高优先级的指令总是优先运行。

+ terminal： 这个参数用来告诉AngularJS停止运行当前元素上比本指令优先级低的指令。但同当前指令优先级相同的指令还是会被执行。
+ template： 参数是可选的，必须被设置为以下两种形式之一：
  + 一段HTML文本；
  + 一个可以接受两个参数的函数，参数为tElement和tAttrs，并返回一个代表模板的字符串。tElement和tAttrs中的t代表template，是相对于instance的。在讨论链接和编译设置时会详细介绍，模板元素或属性与实例元素或属性之间的区别。如果模板字符串中含有多个DOM元素，或者只由一个单独的文本节点构成，那它必须被包含在一个父元素内。换句话说，必须存在一个根DOM元素：

+ templateUrl是可选的参数，可以是以下类型：
  + 一个代表外部HTML文件路径的字符串；
  + 一个可以接受两个参数的函数，参数为tElement和tAttrs，并返回一个外部HTML文件路径的字符串。
无论哪种方式，模板的URL都将通过AngularJS内置的安全层，特别是$getTrustedResourceUrl，这样可以保护模板不会被不信任的源加载。
模板加载后，AngularJS会将它默认缓存到$templateCache服务中。在实际生产中，可以提前将模板缓存到一个定义模板的JavaScript文件中，这样就不需要通过XHR来加载模板了。

+ replace是一个可选参数，如果设置了这个参数，值必须为true，因为默认值为false。默认值意味着模板会被当作子元素插入到调用此指令的元素内部。
  
+ scope：指令在创建的的时候，自带作用域属性（scope），默认情况下为false。如果scope想独立出来，有两种选择，scope为true，或对象{}。

scope属性可以有三个值：
    + false（默认值）：继承父作用域的属性及方法（双向的）。表明不创建新作用域，一切继承父作用域，当scope中的属性改变时，父作用域中对应的属性值会相应改变；
    + true：指令创建一个新的作用域，这个作用域继承了父作用域的原型(单向的）。新创建的作用域，单向继承父作用域属性方法，新作用域中属性值改变时父作用域中对应的属性值不会跟随变化；
    + 对象： 指令创建一个新的独立作用域。如果这样做了，指令的模板就无法访问外部作用域了。你可以使用 @, =, & 这三种绑定策略来指定如何从父作用域中读取和写入数据。

下面是一个AngularJS指令的例子，它使用scope: true创建一个新的作用域：
```javascript
angular.module('myApp', [])
    .directive('myDirective', function() {
        return {
            restrict: 'E',
            scope: true,
            template: '<div>{{ localVar }}</div>',
            link: function(scope) {
                scope.localVar = 'I am local';
            }
        };
    });
```
在这个例子中，myDirective创建了一个新的作用域，并且通过template属性使用了这个作用域中的localVar变量。在链接函数中，我们设置了localVar的值。当这个指令被使用时，它会显示"I am local"。

## 绑定策略

AngularJS提供了几种方法能够将指令内部的隔离作用域，同指令外部的作用域进行数据绑定。
  + @  单项数据绑定，使用@符号将本地作用域同DOM属性的值进行绑定。指令内部作用域可以使用外部作用域的变量
  + = 双向绑定，通过=可以将本地作用域上的属性同父级作用域上的属性进行双向的数据绑定。就像普通的数据绑定一样，本地属性会反映出父数据模型中所发生的改变。
  + &  回调，通过&符号可以对父级作用域进行绑定，以便在其中运行函数。意味着对这个值进行设置时会生成一个指向父级作用域的包装函数。

+ transclude：是一个可选的参数。如果设置了，其值必须为true，它的默认值是false。当你希望创建一个可以包含任意内容的指令时，才使用transclude: true。
例如，假设我们想创建一个包括标题和少量HTML内容的侧边栏，如下所示：
```htm
<div sideboxtitle="Links">
 <ul>
 <li>First link</li>
 <li>Second link</li>
 </ul>
</div> 
```
为这个侧边栏创建一个简单的指令，并将transclude参数设置为true：
```javascript
angular.module('myApp', [])
    .directive('sidebox', function() {
        return {
            restrict: 'EA',
            scope: {
                title: '@'
            },
            transclude: true,
            template: '<div class="sidebox">\
            <div class="content">\
            <h2 class="header">{{ title }}</h2>\
            <span class="content" ng-transclude>\
            </span>\
            </div>\
            </div>'
        };
    }); 
```
这段代码告诉AngularJS编译器，将它从DOM元素中获取的内容放到它发现ng-transclude指令的地方。

借助transclusion，我们可以将指令复用到第二个元素上，而无须担心样式和布局的一致性问题。

+ controller参数可以是一个字符串或一个函数。当设置为字符串时，会以字符串的值为名字，来查找注册在应用中的控制器的构造函数：
```javascript
angular.module('myApp', [])
    .directive('myDirective', function() {
        restrict: 'A', // 始终需要
        controller: 'SomeController'
    })

// 应用中其他的地方，可以是同一个文件或被index.html包含的另一个文件
angular.module('myApp')
    .controller('SomeController', function($scope, $element, $attrs, $transclude) {
        // 控制器逻辑放在这里
    });

//可以在指令内部通过匿名构造函数的方式来定义一个内联的控制器：
angular.module('myApp',[])
    .directive('myDirective', function() {
        restrict: 'A',
        controller: function($scope, $element, $attrs, $transclude) {
            // 控制器逻辑放在这里
        }
    }); 
```

+ controllerAs参数用来设置控制器的别名，可以以此为名来发布控制器，并且作用域可以访问controllerAs。这样就可以在视图中引用控制器，甚至无需注入$scope。
```javascript
angular.module('myApp')
    .directive('myDirective', function() {
        return {
            restrict: 'A',
            template: '<h4>{{ myController.msg }}</h4>',
            controllerAs: 'myController',
            controller: function() {
                this.msg = "Hello World"
            }
        };
    });
```
+ require参数可以被设置为字符串或数组，字符串代表另外一个指令的名字。require会将控制器注入到其值所指定的指令中，并作为当前指令的链接函数的第四个参数。require参数的值可以用下面的前缀进行修饰
  + ?：如果在当前指令中没有找到所需要的控制器，会将null作为传给link函数的第四个参数。
  + ^：指令会在上游的指令链中查找require参数所指定的控制器。
  + ?^：将前面两个选项的行为组合起来，我们可选择地加载需要的指令并在父指令链中进行查找
  + 没有前缀：如果没有前缀，指令将会在自身所提供的控制器中进行查找，如果没有找到任何控制器（或具有指定名字的指令）就抛出一个错误。

## AngularJS 的生命周期

第一个阶段是编译阶段。在编译阶段，AngularJS会遍历整个HTML文档并根据JavaScript中的指令定义来处理页面上声明的指令。

每一个指令的模板中都可能含有另外一个指令，另外一个指令也可能会有自己的模板。当AngularJS调用HTML文档根部的指令时，会遍历其中所有的模板，模板中也可能包含带有模板的指令。

+ compile：选项可以返回一个对象或函数。如果设置了compile函数，说明我们希望在指令和实时数据被放到DOM中之前进行DOM操作，在这个函数中进行诸如添加和删除节点等DOM操作是安全的。

compile和link选项是互斥的。如果同时设置了这两个选项，那么会把compile所返回的函数当作链接函数，而link选项本身则会被忽略。

+ link：它负责在指令的 DOM 元素被添加到 DOM 树中后执行一些操作。它允许你注册 DOM 监听器、更新 DOM 以及执行其他需要在 DOM 元素准备好之后进行的操作。

链接函数的签名如下：
```javascript
link: function(scope, element, attrs) {
    // 在这里操作DOM
}
```
如果指令定义中有require选项，函数签名中会有第四个参数，代表控制器或者所依赖的指令的控制器。

```javascript
// require 'SomeController',
link: function(scope, element, attrs, SomeController) {
    // 在这里操作DOM，可以访问required指定的控制器
} 
```
  + scope：内部注册监听器的作用域。
  + element：实例元素，指使用此指令的元素。在postLink函数中我们应该只操作此元素的子元素，因为子元素已经被链接过了。
  + attrs：实例属性，是一个由定义在元素上的属性组成的列表，可以在所有指令的链接函数间共享。会以JavaScript对象的形式进行传递。
  + controller：指向require选项定义的控制器。如果没有设置require选项，那么controller参数的值undefined。

控制器在所有的指令间共享，因此指令可以将控制器当作通信通道（公共API）。如果设置了多个require，那么这个参数会是一个由控制器实例组成的数组，而不只是一个单独的控制器。

link 函数的作用
+ DOM 操作：link 函数是执行 DOM 操作的理想位置，比如添加事件监听器、修改元素的样式或类。
+ 注册监听器：你可以使用 `$scope.$watch` 来监听作用域上的变化，并在变化发生时执行某些操作。然而，对于 DOM 相关的监听器（如点击事件），直接在 link 函数中注册它们更为常见。
+ 初始化：link 函数也是执行初始化逻辑的好地方，比如设置元素的初始状态或值。

## ngModel
ngModel是一个用法特殊的指令，它提供更底层的API来处理控制器内的数据。当我们在指令中使用ngModel时能够访问一个特殊的API，这个API用来处理数据绑定、验证、CSS更新等不实际操作DOM的事情。

```javascript
angular.module('myApp')
    .directive('myDirective',function(){
        return {
            require: '?ngModel',
            link: function(scope, ele, attrs, ngModel) {
                if (!ngModel) return;
                // 现在我们的指令中已经有ngModelController的一个实例
            }
        };
    }); 
```

如果不设置require选项，ngModelController就不会被注入到指令中。

注意，这个指令没有隔离作用域。如果给这个指令设置隔离作用域，将导致内部ngModel无法更新外部ngModel的对应值：AngularJS会在本地作用域以外查询值。

为了设置作用域中的视图值，需要调用ngModel.$setViewValue()函数

ngModelController中有几个属性可以用来检查甚至修改视图。
1. $viewValue
2. $modelValue
3. $parsers
4. $formatters
5. $viewChangeListeners
6. $error
7. $pristine
8. $dirty
9. $valid
10. $invalid

## 自定义验证
使用AngularJS可以方便地通过指令添加自定义验证。

```javascript
angular.module('validationExample', [])
    .directive('ensureUnique',function($http) {
        return {
            require: 'ngModel',
            link: function(scope, ele, attrs, c) {
                scope.$watch(attrs.ngModel, function() {
                        $http({
                            method: 'POST',
                            url: '/api/check/' + attrs.ensureUnique,
                            data: {
                                field: attrs.ensureUnique, 
                                valud:scope.ngModel
                            })
                            .success(function(data,status,headers,cfg) {
                                    c.$setValidity('unique', data.isUnique);
                                }).error(function(data,status,headers,cfg) {
                                    c.$setValidity('unique', false);
                                });
                    });
            }
        };
    }); 
```

# 配置 config

在模块的加载阶段，AngularJS会在提供者注册和配置的过程中对模块进行配置。在整个AngularJS的工作流中，这个阶段是唯一能够在应用启动前进行修改的部分。
```javascript
angular.module('myApp', [])
 .config(function($provide) {
 }); 
```
需要特别注意，AngularJS会以这些函数书写和注册的顺序来执行它们。

唯一例外的是constant()方法，这个方法总会在所有配置块之前被执行。

当对模块进行配置时，需要格外注意只有少数几种类型的对象可以被注入到config()函数中：提供者和常量。如果我们将一个服务注入进去，会在真正对其进行配置之前就意外地把服务实例化了。我们只能注入用provider()语法构建的服务，其他的则不行。

也可以定义多个配置块，它们会按照顺序执行，这样就可以将应用不同阶段的配置代码集中
在不同的代码块中。
```javascript
angular.module('myApp', [])
    .config(function($routeProvider) {
        $routeProvider.when('/', {
            controller: 'WelcomeController',
            template: 'views/welcome.html'
        });
    })
    .config(function(ConnectionProvider) {
        ConnectionProvider.setApiKey('SOME_API_KEY');
    }); 
```

# 运行块 run
运行块在注入器创建之后被执行，它是所有AngularJS应用中第一个被执行的方法。运行块是AngularJS中与main方法最接近的概念。

运行块通常用来注册全局的事件监听器。例如，我们会在.run()块中设置路由事件的监听器以及过滤未经授权的请求。

假设我们需要在每次路由发生变化时，都执行一个函数来验证用户的权限，放置这个功能唯
一合理的地方就是run方法：
```javascript
angular.module('myApp', [])
    .run(function($rootScope, AuthService) {
        $rootScope.$on('$routeChangeStart', function(evt, next, current) {
            // 如果用户未登录
            if (!AuthService.userLoggedIn()) { 
                if (next.templateUrl === "login.html") {
                    // 已经转向登录路由因此无需重定向
                } else {
                    $location.path('/login');
                }
            }
        });
    }); 
```

# 路由

## $routeChangeStart
AngularJS在路由变化之前会广播$routeChangeStart事件。在这一步中，路由服务会开始加载路由变化所需要的所有依赖，并且模板和resolve键中的promise也会被resolve。

```javascript
angular.module('myApp', [])
    .run(['$rootScope', '$location', function($rootScope, $location) {
        $rootScope.$on('$routeChangeStart', function(evt, next, current) {
    });
 }]); 
```
`$routeChangeStart`事件带有两个参数：
+ 将要导航到的下一个URL；
+ 路由变化前的URL。

## $routeChangeSuccess
AngularJS会在路由的依赖被加载后广播$routeChangeSuccess事件。
```javascript
angular.module('myApp', [])
    .run(['$rootScope', '$location', function($rootScope, $location) {
        $rootScope.$on('$routeChangeSuccess', function(evt, next, previous) {
    });
 }]);
```
`$routeChangeSuccess`事件带有三个参数：

+ 原始的AngularJS evt对象；
+ 用户当前所处的路由；
+ 上一个路由（如果当前是第一个路由，则为undefined）。

## $routeChangeError
AngularJS会在任何一个promise被拒绝或者失败时广播`$routeChangeError`事件。
```javascript
angular.module('myApp', [])
    .run(function($rootScope, $location) {
        $rootScope.$on('$routeChangeError', function(current, previous, rejection) {
    });
 });
```
`$routeChangeError`事件有三个参数：
+ 当前路由的信息；
+ 上一个路由的信息；
+ 被拒绝的promise的错误信息。

## $routeUpdate
AngularJS在reloadOnSearch属性被设置为false的情况下，重新使用某个控制器的实例时，会广播`$routeUpdate`事件。

# 依赖注入

AngularJS使用`$injetor`（注入器服务）来管理依赖关系的查询和实例化。事实上，`$injetor`负责实例化AngularJS中所有的组件，包括应用的模块、指令和控制器等。在运行时，任何模块启动时`$injetor`都会负责实例化，并将其需要的所有依赖传递进去。

这是一个简单的应用，声明了一个模块和一个控制器：
```javascript
angular.module('myApp', [])
    .factory('greeter', function() {
        return {
            greet: function(msg) {alert(msg);}
        }
    })
    .controller('MyController',
        function($scope, greeter) {
            $scope.sayHello = function() {
                greeter.greet("Hello!");
            };
    }); 
```

当AngularJS实例化这个模块时，会查找greeter并自然而然地把对它的引用传递进去：
```htm
<div ng-app="myApp">
 <div ng-controller="MyController">
 <button ng-click="sayHello()">Hello</button>
 </div>
</div>
```
而在内部，AngularJS的处理过程是下面这样的：
```javascript
// 使用注入器加载应用
var injector = angular.injector(['ng', 'myApp']);
 // 通过注入器加载$controller服务：
var $controller = injector.get('$controller');
var scope = injector.get('$rootScope').$new();
// 加载控制器并传入一个作用域，同AngularJS在运行时做的一样
var MyController = $controller('MyController', {$scope: scope})
```
上面的代码中并没有说明是如何找到greeter的，但是它的确能正常工作，因为`$injector`会负责为我们查找并加载它。

## $injector API 
+ annotate() 方法的返回值是一个由服务名称组成的数组，这些服务会在实例化时被注入到目标函数中。
+ get()方法返回一个服务的实例
+ has()方法返回一个布尔值，在$injector能够从自己的注册列表中找到对应的服务时返回true，否则返回false。
+ instantiate()方法可以创建某个JavaScript类型的实例。它会通过new操作符调用构造函数，并将所有参数都传递给构造函数。
+ invoke()方法会调用方法并从$injector中添加方法参数

# 服务
服务提供了一种能在应用的整个生命周期内保持数据的方法，它能够在控制器之间进行通信，并且能保证数据的一致性。

服务是一个单例对象，在每个应用中只会被实例化一次（被$injector实例化），并且是延迟加载的（需要时才会被创建）。服务提供了把与特定功能相关联的方法集中在一起的接口。

## 注册一个服务
通过将方法设置为服务对象的一个属性来将其暴露给外部。
```javascript
angular.module('myApp.services', [])
    .factory('githubService', function($http) {
        var githubUrl = 'https://api.github.com';
        var runUserRequest = function(username, path) {
            // 从使用JSONP调用Github API的$http服务中返回promise
            return $http({
                method: 'JSONP',
                url: githubUrl + '/users/' +
                username + '/' +
                path + '?callback=JSON_CALLBACK'
            });
        };
        
        // 返回带有一个events函数的服务对象
        return {
            events: function(username) {
                return runUserRequest(username, 'events');
            }
        };
    }); 
```

githubService中只包含了一个方法，可以在应用的模块中调用。

## 使用服务
可以在控制器、指令、过滤器或另外一个服务中通过依赖声明的方式来使用服务。AngularJS会像平时一样在运行期自动处理实例化和依赖加载的相关事宜。

将服务的名字当作参数传递给控制器函数，可以将服务注入到控制器中。当服务成为了某个控制器的依赖，就可以在控制器中调用任何定义在这个服务对象上的方法。
```javascript
angular.module('myApp', ['myApp.services'])
    .controller('ServiceController', function($scope, githubService) {
        // 我们可以调用对象的事件函数
        $scope.events = githubService.events('auser');
    });
```
githubService服务已经被注入到ServiceController中，可以像使用任何其他服务一样使用它。

## 创建服务时的设置项
在AngularJS应用中，factory()方法是用来注册服务的最常规方式，同时还有其他一些API可以在特定情况下帮助我们减少代码量。

共有5种方法用来创建服务：
+ factory()
+ service()
+ constant()
+ value()
+ provider()

## factory()
如前所见，factory()方法是创建和配置服务的最快捷方式。

factory()函数可以接受两个参数。
+ name（字符串）需要注册的服务名。
+ getFn（函数）这个函数会在AngularJS创建服务实例时被调用。

```javascript
angular.module('myApp')
.factory('myService', function() {
    return {
        'username': 'auser'
    };
});
```
因为服务是单例对象，getFn在应用的生命周期内只会被调用一次。同其他AngularJS的服务一样，在定义服务时，getFn可以接受一个包含可被注入对象的数组或函数。

## service()
使用service()可以注册一个支持构造函数的服务，它允许我们为服务对象注册一个构造函数。

service()方法接受两个参数。
+ name（字符串）要注册的服务名称。
+ constructor（函数）构造函数，我们调用它来实例化服务对象。

service()函数会在创建实例时通过new关键字来实例化服务对象。

```javascript
var Person = function($http) {
    this.getName = function() {
        return $http({ method: 'GET', url: '/api/user'});
    };
};
angular.service('personService', Person);
```

## provider()
所有服务工厂都是由`$provide`服务创建的，`$provide`服务负责在运行时初始化这些提供者。提供者是一个具有`$get()`方法的对象，`$injector`通过调用`$get`方法创建服务实例。

`$provider`提供了数个不同的API用于创建服务，每个方法都有各自的特殊用途。所有创建服务的方法都构建在provider方法之上。provider()方法负责在`$providerCache`中注册服务。

从技术上说，当我们假定传入的函数就是`$get()`时，factory()函数就是用provider()方法注册服务的简略形式。

下面两种方法的作用完全一样，并且会创建同一个服务。

```javascript
angular.module('myApp')
    .factory('myService', function() {
        return {
            'username': 'auser'
        };
    })
    // 这与上面工厂的用法等价
    .provider('myService', {
        $get: function() {
            return {
                'username': 'auser'
            };
        }
    });
```
是否可以一直使用.factory()方法来代替.provider()呢？

答案取决于是否需要用AngularJS的.config()函数来对.provider()方法返回的服务进行额外的扩展配置。同其他创建服务的方法不同，config()方法可以被注入特殊的参数。

比如我们希望在应用启动前配置githubService的URL：

```javascript
// 使用`.provider`注册该服务
angular.module('myApp', [])
    .provider('githubService', function($http) {
        // 默认的，私有状态
        var githubUrl = 'https://github.com'
        setGithubUrl: function(url) {
            // 通过.config改变默认属性
            if (url) { githubUrl = url }
        }，
        method: JSONP, // 如果需要，可以重写
        $get: function($http) {
            self = this;
            return $http({ method: self.method, url: githubUrl + '/events'});
        }
    });
```
通过使用.provider()方法，可以在多个应用使用同一个服务时获得更强的扩展性，特别是在不同应用或开源社区之间共享服务时。

在上面的例子中，provider()方法在文本githubService后添加Provider生成了一个新的提供者，githubServiceProvider可以被注入到config()函数中。

```javascript
angular.module('myApp', [])
    .config(function(githubServiceProvider) {
        githubServiceProvider.setGithubUrl("git@github.com");
    });
```
如果希望在config()函数中可以对服务进行配置，必须用provider()来定义服务。

provider()方法为服务注册提供者。可以接受两个参数。
+ name（字符串）name参数在providerCache中是注册的名字。name+Provider会成为服务的提供者。同时name
也是服务实例的名字。例如，如果定义了一个githubService，那它的提供者就是githubServiceProvider。
+ aProvider（对象/函数/数组）
  + aProvider可以是多种形式。
  + 如果aProvider是函数，那么它会通过依赖注入被调用，并且负责通过$get方法返回一个对象。
  + 如果aProvider是数组，会被当做一个带有行内依赖注入声明的函数来处理。数组的最后一个元素应该是函数，可以返回一个带有$get方法的对象。
  + 如果aProvider是对象，它应该带有$get方法。

用这个方法创建服务，必须返回一个定义有$get()函数的对象，否则会导致错误。

## constant()
可以将一个已经存在的变量值注册为服务，并将其注入到应用的其他部分当中。例如，假设我们需要给后端服务一个apiKey，可以用constant()将其当作常量保存下来。

constant()函数可以接受两个参数。
+ name（字符串）需要注册的常量的名字。
+ value（常量）需要注册的常量的值（值或者对象）。

constant()方法返回一个注册后的服务实例。
```javascript

angular.module('myApp') .constant('apiKey','123123123')

//这个常量服务可以像其他服务一样被注入到配置函数中：
angular.module('myApp')
    .controller('MyController', function($scope, apiKey) {
        // 可以像上面一样用apiKey作为常量
        // 用123123123作为字符串的值
        $scope.apiKey = apiKey;
    }); 
```

## value()
如果服务的$get方法返回的是一个常量，那就没要必要定义一个包含复杂功能的完整服务，可以通过value()函数方便地注册服务。

value()方法可以接受两个参数。
+ name（字符串）同样是需要注册的服务名。
+ value（值）将这个值将作为可以注入的实例返回。

value()方法返回以name参数的值为名称的注册后的服务实例。

```javascript
angular.module('myApp').value('apiKey','123123123');
```

## 何时使用value()和constant()
value()方法和constant()方法之间最主要的区别是，常量可以注入到配置函数中，而值不行。通常情况下，可以通过value()来注册服务对象或函数，用constant()来配置数据。

```javascript
angular.module('myApp', [])
    .constant('apiKey', '123123123')
    .config(function(apiKey) {
        // 在这里apiKey将被赋值为123123123
        // 就像上面设置的那样
    })
    .value('FBid','231231231')
    .config(function(FBid) {
        // 这将抛出一个错误，未知的provider: FBid
        // 因为在config函数内部无法访问这个值
    });
```

# 架构

## 控制器与指令

作用域的蔓延是Angular框架最令人困惑的一个方面，我们在控制器的`$scope`中定义了很多功能。编写Web应用时，有时会发现控制器的大小逐渐严重失控。

可以移出处理DOM的方法，以减少控制器的大小。把功能移动到自定义指令中，大幅降低了为判断是否公开特定视图或者格式化一个值的需要。

因为我们在视图里绑定了`$scope`上的值，控制器没有必要负责持有DOM对象所需的状态值。比如，我们可以删除控制器中处理显示值的方法。

假设我们有一个登录页面，根据用户的选择的状态显示登录表单或者注册表单，把这个页面命名为showLoginForm：
```javascript
angular.module('myApp', [])
    .controller('LoginController', function($scope) {
        // 如果为true，显示为登录表单
        // 如果为false，显示为注册表单
        $scope.showLoginForm = true;
        $scope.sendLogin = function() {}
        $scope.sendRegister = function() {}
    });
```
在我们的HTML里面，可能以如下方式设置LoginController的功能：
```htm
<div ng-show="showLoginForm">
    <form ng-submit="runLogin()"></form>
</div>
<div ng-show="!showLoginForm">
    <form ng-submit="runRegister()"></form>
</div>
```
尽管这个例子很普通（在我们的`$scope`里只有一个多余的变量），当我们的视图变得越来越复杂时，这种变量的数量会按指数级增长。

使用指令，我们可以把这个值去掉。例如：
```javascript
angular.module('myApp', [])
    .directive('loginForm', function() {
        return {
            scope: {
                onLogin: '&',
                onRegister: '&'
            },
            templateUrl: '/templates/loginRegForms.html',
            link: function(scope, ele, attrs) {
                scope.showLoginForm = true;
                scope.submitLogin = function() {
                    scope.onLogin({user: scope.loginUser});
                }
                scope.submitRegister = function() {
                    scope.onRegister({user: scope.newUser});
                }
            }
        }
    });
 angular.module('myApp', [])
    .controller('LoginController', function($scope) {
        $scope.sendLogin = function() {} 
        $scope.sendRegister = function() {}
    }); 
```

我们可以在视图里调用这个指令，就像调用其他指令那样：
```htm
<div login-form
 on-login="sendLogin(user)"
 on-register="sendRegister(user)"></div>
```
我们的视图变量安全地藏到指令里去了，不再需要在控制器里持有视图条件了。让控制器精简是最佳实践，使用指令能让我们有效地做到这一点。此外，像上面那样把登录的路由隔离到指令中，也使得测试它们的功能更容易。


# digest循环和$apply

digest循环（或更准确地称为脏值检查机制）与Angular的数据绑定和DOM更新机制紧密相关。AngularJS通过脏值检查机制来确保数据模型和视图之间的同步。每当Angular认为可能有模型变化时，它就会启动一个digest循环。在这个循环中，Angular会遍历所有的watchers（观察者），这些watchers是注册在AngularJS中的作用域（scope）上的，用于检测模型的变化。

+ 工作原理：digest循环会检查每个watcher的当前值和旧值是否相同。如果不同，则认为模型发生了变化，并且需要更新视图。更新后，digest循环会再次运行，直到没有检测到任何变化（即所有watchers的当前值和旧值都相同），这时digest循环才会停止。
  
+ 触发时机：digest循环可以由多种事件触发，包括用户输入（如点击、键盘事件等）、Ajax请求回调、定时器等。但是，并不是所有的JavaScript代码都会自动触发digest循环。它不需要开发者显式调用，而是由AngularJS框架在适当的时机自动执行。
  
+ 作用范围：默认情况下，$digest循环只会从调用它的作用域开始，向下遍历其子作用域，检查并更新这些作用域中的watchers。这意味着，如果父作用域中的数据变化没有影响到子作用域中的watchers，那么这些子作用域将不会被检查。

+ 视图更新：一旦$digest循环完成（即模型被认为已经稳定），AngularJS会更新与模型变化相关的DOM元素，以反映最新的数据。

$digest循环有两个主要组成部分：
+ $watch列表；
+ $evalAsync列表。

## $watch 列表
`$watch`列表实际上是一个由watchers组成的数组，每个watcher都负责监视AngularJS作用域（scope）中某个表达式或函数的变化。

每个watcher都包含以下三个重要属性：

1. watchExpression：一个字符串或函数，AngularJS会对其进行求值。如果它是一个字符串，AngularJS会使用$eval函数来求值；如果它是一个函数，则直接调用该函数。这个表达式或函数的结果将被用来与上一个值进行比较，以检测变化。

2. listener：一个回调函数，当watchExpression的结果发生变化时，AngularJS会调用这个函数。它接收三个参数：newValue（新值）、oldValue（旧值）和scope（当前作用域）。

3. last：用于存储watchExpression的上一个求值结果，以便与当前结果进行比较。

## $watch
`$watch `是一个非常重要的API，它允许你监视（监听）Scope上数据模型的变化。

基本用法
`$watch` 函数通常被调用在控制器的上下文中，并接收三个参数：

1. watchExpression（监视的表达式）: 这是一个字符串或者一个函数，AngularJS会计算它的值并与上一次的值进行比较。如果值发生了变化，那么就会执行回调函数。

2. listener（回调函数）: 当监视的表达式发生变化时，这个函数会被调用。AngularJS会向这个函数传递两个参数：新值和旧值。

3. objectEquality（可选）：一个布尔值，用于指定是否进行深度监测。如果设置为true，则AngularJS会检查对象中的每个属性是否发生变化。

在AngularJS中，你可以使用`$scope.$watch`方法在作用域上注册一个watcher。例如：

```javascript
app.controller('MyController', function($scope) {  
    $scope.myVar = 'Hello, World!';  
  
    // 使用$watch来监视myVar的变化  
    var unregisterWatch = $scope.$watch('myVar', function(newValue, oldValue) {  
        console.log('myVar changed from ' + oldValue + ' to ' + newValue);  
    });  
  
    // 假设在某个时刻，你改变了myVar的值  
    $scope.$apply(function() {  
        $scope.myVar = 'Goodbye, World!';  
    });  
});
```

当myVar的值从'Hello, World!'变化到'Goodbye, World!'时，$watch的回调函数会被触发，并在控制台中打印出旧值和新值。

`$watch`函数会给监听器返回一个注销函数，我们可以调用这个注销函数(unregisterWatch)来取消Angular对当前值的监控。

## $watchCollection
`$watchCollection`是`$scope`对象上的一个方法，它提供了一种监视（监听）集合（如数组或对象）中项变化的方式。与`$watch`相比，`$watchCollection`对于监视复杂数据结构的变化更为高效和适用。

`$watchCollection` 的特点
1. 深度监听对象属性：对于对象，`$watchCollection`会监听对象内部属性的添加或删除，以及属性值的变化。但它不会进行深度的属性值比较（即不会深入对象的嵌套属性中）。

2. 数组的特殊处理：对于数组，`$watchCollection`会监听数组内部元素的添加、删除或整个数组的替换。但它不会监听数组内部元素属性的变化（除非该元素本身是一个对象，并且该对象的属性被添加、删除或整个对象被替换）。

3. 性能优化：由于`$watchCollection`只关注集合的结构变化（如元素的添加、删除或替换），而不是深入比较每个属性的值，因此它在处理大型或复杂的数据结构时比`$watch`（特别是使用深度比较时）更高效。


假设你有一个AngularJS应用，其中包含一个对象person，你想要监视这个对象中name和age属性的变化：

```javascript
app.controller('MyController', function($scope) {  
    $scope.person = { name: 'Alice', age: 30 };  
  
    // 使用$watch来监视person对象中的name和age属性（不推荐，因为不够高效）  
    // $scope.$watch('person', function(newValue, oldValue) {  
    //     // 这里会触发很多次，即使只有一个属性变化  
    //     console.log('Person changed:', newValue, oldValue);  
    // }, true); // 注意：这里的true表示进行深度比较  
  
    // 使用$watchCollection来监视person对象（更推荐）  
    $scope.$watchCollection('person', function(newValue, oldValue) {  
        // 这里只会在person对象的结构发生变化时触发  
        // 比如添加或删除了属性，或者整个对象被替换  
        console.log('Person collection changed:', newValue, oldValue);  
    });  
  
    // 注意：如果只想监视person对象的特定属性（如name或age），则应该使用单独的$watch  
    // 例如：$scope.$watch('person.name', function(newValue, oldValue) { ... });  
});
```

然而，需要注意的是，上面的`$watchCollection`示例实际上并不会如你所期望的那样工作，因为它只会在person对象的结构发生变化时触发（如添加、删除属性或整个对象被替换），而不会单独监视name或age属性的变化。如果你想要监视person对象中name或age属性的单独变化，你应该为这些属性分别设置$watch。

`$watchCollection`的真正用途是当你需要监视一个数组或对象集合的整体结构变化时，而不是深入比较集合中每个元素或属性的值。如果你需要深入比较对象属性的值，你应该使用`$watch`并设置其第三个参数为true来进行深度比较。但是，请注意这可能会影响应用的性能，特别是在处理大型或复杂的数据结构时。

## $evalAsync 列表

在AngularJS中，`$evalAsync` 是一个 `$scope` 的方法，用于在当前的 `$digest` 循环结束之前或在下一个 `$digest` 循环开始时执行表达式。这个方法非常有用，因为它允许你安排代码在AngularJS的脏检查机制之外执行，但仍然能够确保在视图更新之前执行。

然而，需要注意的是，`$evalAsync` 本身并不维护一个显式的“列表”或队列，而是将任务添加到AngularJS的内部任务队列中。这个队列由AngularJS管理，并在适当的时机执行队列中的任务。

`$evalAsync` 的工作原理
1. 添加任务：当你调用 `$scope.$evalAsync(expression)` 时，AngularJS会将提供的表达式或函数添加到其内部的任务队列中。

2. 执行时机：如果当前正在执行` $digest` 循环，则 `$evalAsync` 安排的任务将在当前 `$digest` 循环的末尾执行。如果当前没有执行 `$digest` 循环，则AngularJS会尽快启动一个新的 `$digest` 循环来执行这些任务。

3. 执行任务：在 `$digest` 循环的适当阶段，AngularJS会遍历 `$evalAsync` 队列并执行其中的任务。

4. 更新视图：一旦所有 `$evalAsync` 任务都执行完毕，AngularJS将更新视图以反映数据模型中的任何变化。

5. 注意事项: 由于 `$evalAsync` 是在AngularJS的脏检查机制之外工作的，因此它不会触发额外的 `$digest` 循环（除非它本身是在一个 `$digest` 循环之外被调用的）。`$evalAsync` 是处理与AngularJS数据绑定相关的异步操作（如从服务器获取数据后更新视图）的一种高效方式。

6. 替代方案: 对于与AngularJS不直接相关的异步操作（如从服务器获取数据），你可能需要使用 `$timeout `或 `$http` 等服务，这些服务会处理必要的` $digest `循环触发。

## $apply

在AngularJS中，`$apply() `是一个 `$scope`（或其子对象）的方法，它用于在AngularJS框架外部执行的代码中启动一个 `$digest` 循环。`$apply()` 方法接受一个函数作为参数，并立即执行这个函数。之后，它会触发一个 `$digest` 循环来检查并更新视图，以反映数据模型中的任何变化。

为什么需要 `$apply()`

在AngularJS中，数据绑定和视图更新是自动的，但这通常只发生在AngularJS框架内部。当你在AngularJS框架外部（例如，在原生JavaScript的 setTimeout、setInterval 回调中，或者在第三方库的事件处理程序中）修改AngularJS的数据模型时，AngularJS不会自动知道这些变化，因此不会自动更新视图。

这就是 `$apply()` 方法发挥作用的地方。通过在 `$apply()` 方法中包裹你的代码，你可以告诉AngularJS：“嘿，我在这里做了一些改变，你可能需要检查一下并更新视图。”

如何使用 `$apply()`
```javascript
app.controller('MyController', function($scope) {  
    $scope.counter = 0;  
  
    // 使用setTimeout在AngularJS框架外部修改数据模型  
    setTimeout(function() {  
        $scope.$apply(function() {  
            // 这里是AngularJS框架外部的代码  
            $scope.counter = $scope.counter + 1;  
            // AngularJS现在会知道counter的值已经改变，并更新视图  
        });  
    }, 1000);  
  
    // 或者，如果你只需要执行一个简单的表达式，可以直接传递给$apply  
    // 但请注意，这样做可能会隐藏潜在的错误（如表达式中的错误），因为它不会立即抛出  
    // setTimeout(function() {  
    //     $scope.$apply('$scope.counter = $scope.counter + 1');  
    // }, 1000);  
});
```
注意事项

当调用`$apply`时，它会触发整个应用（或至少是从`$rootScope`开始）的`$digest`循环。这意味着，无论数据变化发生在哪个作用域中，只要它影响了某个watcher，那么整个应用中的相关watchers都会被检查并更新。因此，从性能角度考虑，如果开发者能够确定数据变化的影响范围，并只在该范围内调用`$digest`（而不是`$apply`），则可以提高应用的性能。

过度使用 `$apply() `或在不必要的情况下使用它可能会降低应用的性能，因为每个 `$apply()` 调用都会导致 `$digest` 循环的执行。如果在 `$digest` 循环已经在进行中的时候调用 `$apply()`，AngularJS会抛出一个异常。

为了避免这种情况，你可以使用 `$scope.$applyAsync()`（如果可用），`$scope.$applyAsync()` 是 AngularJS 中的一个方法，它用于在当前的 `$digest` 循环结束后或下一个 `$digest` 循环开始时异步地执行一个表达式或函数。与 `$scope.$apply()` 相比，`$applyAsync()` 的主要优势是它不会立即触发 `$digest` 循环，从而减少了在 `$digest` 循环已经在进行中时调用 `$apply()` 可能导致的冲突和性能问题。

`$applyAsync() `并不保证你的代码会立即执行。它只保证你的代码会在 AngularJS 的下一个 `$digest` 循环中执行。

具体来说，AngularJS框架外部的代码可能包括：

1. 原生JavaScript事件处理器：这些是在HTML元素上直接通过addEventListener或HTML属性（如onclick）设置的函数，它们不是通过AngularJS的ng-click等指令定义的。

2. 第三方库和框架的回调：当你使用jQuery、D3.js、React或其他非AngularJS的JavaScript库时，这些库可能会提供回调函数或事件监听器，这些函数在AngularJS的控制之外执行。

3. 定时器（setTimeout和setInterval）：使用这些原生JavaScript函数设置的回调函数将在AngularJS的脏检查循环之外执行。这意味着，如果你在这些回调函数中更改了AngularJS的模型数据，你需要手动告诉AngularJS更新视图，这通常是通过调用`$scope.$apply()`来实现的。

4. Web Workers：Web Workers允许你在与主线程分离的后台线程中运行脚本。由于这些脚本在主线程的控制之外运行，因此它们也被视为AngularJS框架外部的代码。

5. 服务器通信回调：虽然AngularJS提供了$http服务来处理HTTP请求，但如果你使用了XMLHttpRequest或Fetch API（没有通过AngularJS的包装器）来发送请求，并且在请求完成时设置了回调函数，则这些回调函数也是在AngularJS框架外部执行的。

6. 自定义的全局或模块级JavaScript函数：这些函数可能不是作为AngularJS应用的一部分编写的，或者它们可能被设计为在AngularJS应用的上下文中以非AngularJS特定的方式使用。

## $applyAsync() 与 $evalAsync()

选择哪个函数取决于你的具体需求和场景。如果你不确定，通常 `$applyAsync()` 是一个更安全、更通用的选择，因为它不会尝试在当前 `$digest` 循环中插入代码，从而减少了潜在的冲突和错误。

`$evalAsync()` 的真正价值在于它在 AngularJS 的生命周期中的灵活性，允许你在几乎任何需要立即但异步地更新视图的地方使用它，而无需担心 `$digest` 循环的当前状态。
