---
layout: post
title:  "npm scripts"
date:   2016-04-04 
---

`npm scripts`是`package.json`中的一个配置属性。做单元测试时，常常会用到`npm test`，其实`npm scripts`能做的更多。

最近社区有开发者提到，用`npm scripts`替换现有的构建工具：grunt、gulp等，由此可见`npm scripts`是一个强大的工具。

### 什么是 npm scripts
先来看一个简单的`package.json`(截取了`node-sass`的`package.json`中的部分属性)

```
{
	"name": "node-sass",
	"version": "3.5.0-beta.1",
	"scripts": {
		"coverage": "node scripts/coverage.js",
		"install": "node script/install.js",
		"postinstall": "node scripts/build.js",
		"pretest": "node_modules/.bin/jshint bin lib scripts test",
		"test": "node_modules/.bin/mocha test",
		"build": "node scripts/build.js --force",
		"prepublish": "not-in-install && node scripts/prepublish.js || in-install"
	}
}
```

`npm` 会有一些默认的命令，包括但不限于：publish、test、install等。
执行这些命令时，实际执行的是`scripts`中对应属性的脚本。比如，npm install 时，会执行`node script/install.js`。同时提供了相对应的勾子：`pre`和`post`，`preinstall` 和 `postinstall` 会相应的在 install 前后执行。


### npm scripts 能干些什么
了解什么是`npm scripts`后，它能做的事情就很明确了：`在scripts中定义命令（执行一个node脚本，或者有全局的node包去跑一些命令），然后用npm的命令行执行对应的命令`。默认的一些指令的作用已经很明确了，比如`npm test`会执行`test`对应的脚本等。那么如何自定义脚本呢？


```
"scripts": {
	"dev": "node-sass src/style.scss dest/style.css"
}
```

上面的 demo 中定义了一个 dev 脚本，运行 `npm run dev` 即可执行对应的命令。

### npm script 替代构建
现有的构建体系依赖的插件体系，都是用构建体系本身去封装第3方库，不可避免的会存在以下问题：

1. 第3方库更新了，插件没有更新。
 
2. 报错了，是第3方库的问题还是插件的问题？

3. 文档，文档，文档。
 
4. ......

使用 `npm scripts` 至少可以避免1，2，3。使用者只需要关注于第3方库如何使用，无须关心对应的插件用法。同时，不用构建体系之间的移植也更加方便。

当然，`npm scripts`使用本身也有些问题：

1. 某个功能如果没有对应的第3方库，编写插件比较麻烦。构建体系本身会提供一些api，编写插件时，可以复用这些api，但是用`npm scripts`时，实际上是编写 node 插件，api的级别更低。

2. 跨平台。 `npm scripts`可以基于管道的方式（一个task的输出，作为另一个task的输入），在windows下支持不友好。


### 如何选择
说了这么多 `npm scripts`的好处，真的就要用它替代 grunt、gulp、fis吗？
**自己选择咯！**

