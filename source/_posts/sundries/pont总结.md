---
title: pont总结
date: 2024-06-07 17:13:53
tags:
---

Pont 把 swagger、rap、dip 等多种接口文档平台，转换成 Pont 元数据。Pont 利用接口元数据，可以高度定制化生成前端接口层代码，接口 mock 平台和接口测试平台。

安装vscode 插件后，vscode经常卡死。初始可以使用默认配置生产Typescript接口。


# pont-config.json

```json
{
  "outDir": "./src/api", // 生成代码的存放路径，使用相对路径即可
  "templatePath": "./pont-template"  // 生成模板文件路径，使用相对路径即可

  // 配置每个数据来源, 一个项目里面往往会有多个数据源，建议这样配置
  "origins": [
    {
      // 接口平台提供数据源的 open api url（需要免登），目前只支持 Swagger。
      "originUrl": "https://petstore.swagger.io/v2/swagger.json",
      // 建议配置，后面可以在模板文件里面使用
      "name": "petstore",
      // 配置是否启用多数据源,建议默认开启
      "usingMultipleOrigins": true
    }
  ],
  // 本地mock配置
  "mocks": {
    // 是否开启本地mock
    "enable": true,
    // 本地mock的接口，注意看看该端口是否被占用
    "port": 8081
  },
  
}
```

https://juejin.cn/post/6963153782075031566

https://juejin.cn/post/7130267185158553637


https://www.cnblogs.com/dingshaohua/p/17329129.html

https://juejin.cn/post/7047106965838757918

