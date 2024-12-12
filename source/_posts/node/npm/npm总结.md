---
title: npm总结
date: 2024-06-14 16:57:03
tags:
---

每天都在用，却不知所以然。浅析npm init、create、exec，npx

# npm exec、npm x、npx 

npm官方文档中指出x，其实就是exec的别名，通俗来讲意思就是npm exec、npm x，两个命令是完全等价的。

npm文档中提到：npx的二进制文件在npm v7.0.0中被重写，而独立的npx包在当时已弃用。npx使用npm exec命令，而不是单独的参数解析器和安装过程。并提供了一些支持，以保持与它在以前版本中接受的参数的向后兼容性。
因此我们姑且理解 npx  = npm exec或npm x

## npm exec <pkg>的执行流程

1. 在本地查找是否有<pkg>对应的npm包
    + 若找到，则运行这个包的package.json中bin字段对应的可执行文件
    + 若未找到，在远程npm仓库查找是否有<pkg>对应的npm包
      + 若找到，则提示是否下载到本地
      + 下载完成后，再运行这个包package.json中bin字段对应的可执行文件

2. 同时，在执行bin字段有几点注意的
   + 如果bin只有一个入口，那么可以执行
   + 如果bin有多个入口，则寻找和包名一样的那个入口
   + 如果没找到，则npm exec <pkg>报错

下面用create-vite这个npm包举个例子：

```json
// 这是他的package.json中的字段，简单列举，省略很多
{
	"name": "create-vite"
	"bin": {
		"create-vite": "index.js",
		"cva": "index.js"
	}
	...
}
```
执行npm exec crate-vite这条命令后

1. 首先本地查找是否有create-vite这个npm包
    + 找到，则运行create-vite这个npm包中的package.json中bin字段对应的可执行文件，即index.js这个文件
    + 若未找到，在远程npm仓库查找是否有create-vite这个包
      + 若找到，则提示是否下载到本地
      + 下载完成后，再运行这个包package.json中bin字段对应的可执行文件，即index.js这个文件

看了bin字段的内容，我们不妨在执行一下npm exec cva这条命令，执行流程和上面的npm exec crate-vite也是一样的，但是执行后我们会发现，这个命令报错了，那么为什么呢？

那么我们来分析一下：执行npm exec cva这条命令后

1. 首先本地查找是否有cva这个npm包
   + 找到，则运行cva这个npm包中的package.json中bin字段对应的可执行文件
   + 若未找到，在远程npm仓库查找是否有cva这个包
     + 若找到，则提示是否下载到本地
     + 下载完成后，再运行这个包package.json中bin字段对应的可执行文件

这样我们应该就明白了，执行npm exec cva这条命令后，我们其实是需要执行cva这个包中package.json文件中的bin字段，而不是create-vite这个包中的package.json文件中的bin字段。

说巧不巧，npm官方库中还真有cva这个包，他的package.json文件如下：
```json
{
  "name": "cva",
  "version": "0.0.0",
  "description": "Awesome node module",
  "keywords": [
    "placeholder",
    "zce"
  ],
  "license": "MIT",
  "author": {
    "name": "zce",
    "email": "w@zce.me",
    "url": "https://zce.me"
  }
}
```
从上面我们可以看出，cva这个包的package.json中根本就没有bin字段，所以npm exec cva显然会报错。


# npm init、npm create、npm innit
npm官方文档中指出create，innit其实就是init的别名，通俗来讲意思就是，其实npm init，npm create，npm innit三个命令是完全等价的。

## 语法

npm init
npm init <initializer>

后面没有<initializer>是用来创建package.json文件的。

这里的npm init <initializer>实际会调用npm exec create-<initializer>, 也相当于npx create-<initializer>。
我们可以把这个<initializer>理解为 有特殊格式包名的包的简称，它真正的包名为create-<initializer>，也只有符合这种特殊格式(create-<xxxx>)的包才可以执行这样的命令。

# npm init、 npm install

1. npm init：
   + 后面没有<initializer>是用来创建项目package.json
   + 后面有<initializer>实际会调用npm exec create-<initializer>
   + npm init --yes或npm init -y:从当前目录中提取的信息生成默认的package.json，创建过程中不会提问

2. npm install 安装本地包，安装package.json文件中的依赖包到并且生成node_modules文件夹。

