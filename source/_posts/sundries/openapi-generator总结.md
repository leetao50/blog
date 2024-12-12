---
title: openapi-generator总结
date: 2024-06-13 15:31:25
tags:
---

Github地址：https://github.com/OpenAPITools/openapi-generator

官网文档： https://openapi-generator.tech/docs/installation



在前端开发时候，总是会有许多的接口需要你去对接。如果手动去添加一些接口，那会有许多的工作量，这些工作都是重复且无趣的。在一番搜索下，找到了一个不错的方案。通过一个 openapi 规范的接口 JSON 去自动生成接口文件，并在前端调用。

# 什么是OpenAPI

OpenAPI是一种用于指定REST API的格式。它也被称为“Swager规范”。格式记录如下：
https://swagger.io/specification/

安装 OpenAPI Generator
`npm install @openapitools/openapi-generator-cli`

安装npm后，应该在不带参数的情况下运行其二进制文件。

`openapi-generator-cli`

当您第一次运行npm二进制文件时，它将选择Java程序的稳定版本并下载它。

npm将把Java程序的版本号放在一个名为openapitools.json的新文件中。您应该将此文件添加到版本控制中。

# 运行 Code Generator

一旦安装了npm并生成了配置文件openapitools.json，我们就可以生成代码了。使用的命令如下：

```cmd
npx @openapitools/openapi-generator-cli generate --skip-validate-spec --additional-properties=apiPackage=api,modelPackage=model,withSeparateModelsAndApi=true -i http://localhost:8081//v2/api-docs -g typescript-axios -o src\api\auth   
```

该命令使用以下选项：

+ -i 指定Swager规范文件
+ -g 指定生成文件输出格式
+ -o 指定输出目录
+ --additional-properties 指定特定于所选生成文件输出格式的一些选项
+ --skip-validate-spec 

我喜欢将脚本添加到我的 package.json文件的scripts部分，它允许我们通过 npm run genauth 从 npm 运行代码生成器。脚本如下：

```shell
"cleanauth":"rimraf  src\api\auth",
"genauth": "npx @openapitools/openapi-generator-cli generate --skip-validate-spec --additional-properties=apiPackage=api,modelPackage=model,withSeparateModelsAndApi=true -i http://localhost:8081//v2/api-docs -g typescript-axios -o src\api\auth"
```

![图 0](../cc1df421beae646c9bc1897a93138273c02f6af5d2f5abe0e12a1414f2799a93.png)  


|-api
    |-account-controller-api.ts
    |-xxx-api.ts
|-model
    |-account.ts
    |-index.ts
    |-xxx.ts
|-api.ts
|-base.ts
|-common.ts
|-configuration.ts

api:存储访问服务(xxx-api)的实现
api.ts:导出api目录中生成的xxx-api服务
base.ts:存储 Axios 实例及配置方法
common.ts: 存储配置 Axios实例的方法
configuration.ts:存储 Configuration 类型
model:存储根据Swager规范生成的 实体类型


在src/api目录下，创建 index.ts
```javascript
//这里用的是vite来创建项目
//从环境配置里获取url
const envUrl = `${import.meta.env.VITE_BASE_URL}`;

export const accountApi = new AccountControllerApi({
    basePath: envUrl,
    isJsonMime(mime) {
      return mime === "application/json";
    },
});

```

这样就实现了自动化生成接口文档，并且可以配置对应的 token 和请求路径。
