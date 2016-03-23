---
layout: post
title:  "如何编写fis3插件"
date:   2015-09-25
---
目前业务正在逐步迁移到fis3和lego，有许多和业务相关的fis插件需要处理。

### fis 编译流程

![fis-compile-flow](https://cloud.githubusercontent.com/assets/3880323/10098419/c00bd326-63b3-11e5-9c15-b9bbf3eb27a0.png)
官方的这张图，对fis的构建流程讲述的很清楚了，主要包括单文件编译和打包，业务中的插件也主要是这两种，至于是pre还是post，差别不是特别大。主要记住一点区别：**单文件编译阶段，无法获取文件之间的关联信息；打包阶段，能够拿到项目的所有文件**

### iconfont 实例

本文以fis接入iconfont插件为例，讲述iconfont接入方案，iconfont大致的流程：

1. 同步svg，将项目用到的svg，通过[iconfont平台](http://iconfont.imweb.io)同步到项目目录

2. 编译svg，生成字体文件

3. 接入字体相关的css问题

4. html引入css文件

### 单文件编译处理iconfont

大体的逻辑是：

1. 遍历项目目录下的所有svg，生成字体文件

2. 生成css

3. 所有业务html引入css

配置：

```
    fis.match('/*.html', {
         preprocessor: fis.plugin('iconfont',  {
             svgPath: '../svgs',
             output: 'modules/common/fonts'
         })
    })
```
对根目录下的所有html应用组件，编译一次svg，生成字体文件，然后对业务html引入css文件。

### 打包阶段处理iconfont

上面的处理方式很粗暴，svg并没有按需生成，更好的方式是：

1. 获取项目中用到的icon

2. 查找对应的svg，生成字体文件

3. 字体css生成

4. html引入css

其中，第1步，需要通过编译所有文件获取结果，所以必须要在打包截断处理。参考[这里](https://github.com/haledeng/fis-postpackager-iconfont)


### 单元测试

fis3一个很大的优势是提供了单元测试的接口，写完插件后，记得要用单元测试过下，同时这也是一个调试的过程（fis2的调试只能依赖具体的项目）,主体的代码架构是:

```
var fs = require('fs'),
    path = require('path'),
    fis = require('fis3'),
    _ = fis.util,
    expect = require('chai').expect,
   // 下面这两个库是提供测试的关键库，用于release
    _release = fis.require('command-release/lib/release.js'),
    _deploy = fis.require('command-release/lib/deploy.js'),
   // 自己编写的插件入口文件
    iconfont = require('../src/');

function release(opts, cb) {
    opts = opts || {};
    _release(opts, function(error, info) {
        _deploy(info, cb);
    });
}
describe('fis-postpackager-iconfont', function() {
    var root = path.join(__dirname, 'src');

    fis.project.setProjectRoot(root);
    // 测试用例

    beforeEach(function() {
          // 这里写各种fis的配置
           fis .match('*', {
                deploy: iconfont({
                         // 自己插件的配置
                })
            });
    });

    it('test case', function() {

        release({
            unique: true
        }, function() {
            // 文件是否生成
            // release 之后，检测是否和预期一致
            console.log('release end');
        });
    });
});
```

### npm publish

这个没啥好说的，注册帐号之后，直接在插件目录下 **npm init**，按照格式填写package.json后，直接运行 **npm publish**，将代码发布出去，这样方便其他人使用。


