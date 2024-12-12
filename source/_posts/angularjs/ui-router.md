---
title: ui-router
date: 2024-08-13 14:28:25
tags:
---
```javascript

angular.module('home', [])
    .config(['$stateProvider', function ($stateProvider) {
        $stateProvider
            .state('nav.home', {
                url: '/home',
                templateUrl: 'views/home/home.html',
                controller: 'homeController',
                withoutAuth: true,
                params: {
                    isFromLogin: null
                },
                resolve: {
                    loadLib: ['$ocLazyLoad', function ($ocLazyLoad) {
                        return $ocLazyLoad.load([
                            'scripts/xxx.js'
                        ]);
                    }]
                },
                modules: {
                    first: '首页',
                    second: '首页'
                }
            })
}]);

```
这段代码是使用AngularJS框架结合ui-router（一个第三方路由库）来定义一个名为home的模块，并配置了一个路由状态nav.home。

下面是这段代码的详细解释：

`angular.module('home', [])：`
创建一个名为home的AngularJS模块。
第二个参数[]表示这个模块不依赖其他模块。
`.config(['$stateProvider', function ($stateProvider) {...}])：`
使用.config方法来配置模块。
注入`$stateProvider`服务，用于定义路由状态。
`$stateProvider.state('nav.home', {...})：`
定义一个名为`nav.home`的路由状态。
`url: '/home'：`当URL匹配`/home`时，将激活这个状态。
`templateUrl: 'views/home/home.html'：`指定这个状态的视图模板文件。
`controller: 'homeController'：`指定这个状态的控制器。
`withoutAuth: true：`可能是一个自定义属性，用于指示这个状态不需要身份验证。
`params: { isFromLogin: null }：`定义这个状态可以接受的参数，isFromLogin是可选的。
`resolve: { loadLib: [...] }：`在路由激活之前，需要加载的依赖项。这里使用$ocLazyLoad来异步加载`scripts/datetime-picker.js`脚本。
`modules: { first: '首页', second: '首页' }：`可能是一个自定义属性，用于定义这个状态在导航或菜单中的显示名称。
总的来说，这段代码定义了一个AngularJS模块，并配置了一个路由状态，用于在应用中展示一个名为“首页”的页面，同时指定了页面的控制器、视图模板、参数和需要加载的脚本。

UI-Router提出了$state的概念。一个$state是一个当前导航和UI的状态，每个$state需要绑定一个URL Pattern。 在控制器和模板中，通过改变$state来进行URL的跳转和路由。

# 路由加载的三种方法
```javascript

//1、·调用$state.go方法   
$state.go('detail', {id: 1});   
$state.go(“index');

//2、点击包含ui-sref指令的链接
<a ui-sref="contacts">Contacts</a>
<a ui-sref="contacts.detail({contactId: 42})">Contact 42</a>
 
//3、url跳转
<a href="#/index" >index</a>
```

# 视图嵌套
ui-router的视图可以嵌套，视图嵌套通常对应着$state的嵌套。 index.detail是index的子$state，index_detail.html也将作为index.html的子页面。ui-view可以配合$state进行任意层级的嵌套， 即index_detail.html中仍然可以包含一个ui-view，它的$state可能是index.detail.hobbies。

# 参数说明：
+ url：默认相对路径（以^开头的是绝对路径）
+ views：每个子视图可以包含自己的模板、控制器和预载入数据。 (后2项选填,控制器可以在view中绑定)
+ abstract：抽象模板不能被激活
+ template: HTML字符串或者返回HTML字符串的函数
+ templateUrl: HTML模板的路径或者返回HTML模板路径的函数
+ templateProvider：返回HTML字符串的函数
+ controller、controllerProvider：指定任何已经被注册的控制器或者一个作为控制器的函数
+ resolve：在路由到达前预载入一系列依赖或者数据，然后注入到控制器中。
+ data：数据不会被注入到控制器中，用途是从父状态传递数据到子状态。
+ onEnter/onExit：进入或者离开当前状态的视图时会调用这两个函数.
+ resolve 为控制器提供可选的依赖注入项。是由 key/value 组成的键值对象。
  + key – {string}：注入控制器的依赖项名称。
  + value - {string|function}：
    + string：一个服务的别名
    + function：函数的返回值将作为依赖注入项，如果函数是一个耗时的操作，那么控制器必须等待该函数执行完成（be resolved）才会被实例化。比如,视图都需要用户登录后才能访问,那么判断是否登录就可以做成一个控制器依赖`resolve: {authentication:['topicAuth', '$q', function(topicAuth, $q){ return $q.when().then(function(){ return topicAuth.authentication(); }); }]}`

# 动态加载模板/内容
在实际使用时往往需要根据参数动态加载不同模板/内容。此时可以通过如下两种方式实现。

```javascript
//1、用参数作为文件名，拼接文件名加载
templateUrl: function ($stateParams){
    return '/topic/detail_' + $stateParams.type + '.html';
}
  
//2、用参数赋值，修改内容
templateProvider: function ($timeout, $stateParams) {
    return $timeout(function () {
        return '<h1>' + $stateParams.topicId + '</h1>'
  }, 100);
}

```

