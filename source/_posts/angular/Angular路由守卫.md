---
title: Angular路由守卫
date: 2023-08-24 10:23:51
tags:
---

我们使用Angular Guards来控制用户是否可以导航到或离开当前路由。

我们使用路由守卫的常见场景之一是身份验证。我们希望我们的应用程序能够阻止未经授权的用户访问受保护的路由。当用户试图导航到受保护的路由时，angular调用CanActivate。然后，我们挂接到CanActivate防护程序，并使用身份验证服务来检查用户是否有权使用路由，如果没有，我们可以将用户重定向到登录页面。

# 守卫路由的作用

+ 确认导航操作
+ 询问是否在离开视图之前进行数据保存
+ 允许特定用户访问应用程序的某些部分
+ 在导航到新路由之前验证路由参数
+ 在显示组件之前获取一些数据

# 路由守卫类型
Angular Router支持五种不同的路由守卫，您可以使用它们来保护路由
+ CanActivate：决定是否可以激活路由（或使用组件）。在用户未被授权导航到目标组件的情况下，此保护非常有用。或者用户可能没有登录到系统
+ CanDeactivate：决定用户是否可以离开组件（导航离开当前组件）。在用户可能有一些未保存的挂起更改的情况下，此路由非常有用。
+ Resolve：延迟路由的激活，直到某些任务完成。在激活路由之前，您可以使用Resolve守卫从后端API获取数据
+ CanLoad：防止加载延迟加载模块。我们通常在不希望未经授权的用户看到模块的源代码时使用此保护。CanLoad与CanActivate类似，但有一点不同。CanActivate防止访问特定的路由。CanLoad防止整个延迟加载模块被下载，从而保护该模块中的所有路由。
+ CanActivateChild：确定是否可以激活子路由。CanActivateChild与CanActivate非常相似。我们将CanActivateChild应用于父路由。每当用户试图导航到其任何子路由时，Angular都会调用CanActivateChild。这使我们能够检查某些情况，并决定是继续导航还是取消导航。

# 如何创建Angular路由守卫

+ 建立Guard服务。
  > @Injectable()
  export class ProductGuardService implements CanActivate {}
+ 在服务中实现Guard方法
  > canActivate(): boolean {    
    // Check weather the route can be activated;    
    return true;     
    // or false if you want to cancel the navigation; 
    }

+ 在根模块中注册Guard服务
  > providers: [ProductService,ProductGuardService]

+ 更新路由以使用Guard服务
  > { path: 'product', component: ProductComponent, canActivate : [ProductGuardService] }

# 守卫路由调用顺序

添加路由守卫语法
```javascript
{ path: 'product', component,        
    canActivate : any[],        
    canActivateChild: any[],       
    canDeactivate: any[],       
    canLoad: any[],       
    resolve: any[] 
}
```

一条路由可以有多个守卫保护，并且可以在路由层次结构的每个级别都有守卫保护。
+ 始终首先检查CanDeactivate（）和CanActivateChild（）防护。检查从最深的子路径开始到顶部。
+ 接下来将检查CanActivate（）保护，并从顶部开始检查最深的子路由。
+ 接下来调用CanLoad（），如果要异步加载功能模块。
+ Resolve（）Guard是最后调用的。

如果任何防护返回false，则Angular路由器将取消导航。

