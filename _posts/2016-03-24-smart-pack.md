---
layout: post
title:  "从smart漫谈打包"
date:   2016-03-24 03:21:03 +0800
categories: fis
---
项目构建迁移到fis体系后，打包的问题就跟着来了。打包的基本方式是：分析依赖，合并文件，解决引入。


### CSS处理方式

#### 来源

+ link 引入
+ require 引入
+ 其它模块的同名依赖

#### 提取来源

+ 正则匹配link
+ require的方式可以直接通过 file.requires获取
+ 处理方式略复杂，需要在分析js依赖时，判断依赖是css文件

#### 打包逻辑

最粗暴的方式是allInOne，全部打成一个包。但是多个页面的业务中，有重复的代码被打到了不同的文件中。这种情况下，打包成2个文件比较合适：**一个基础样式，一个页面样式**。

**common，reset等公共样式打包成一个文件，Page 级别的样式打包成一个文件**，common级的样式，每个页面都引入（加载可以利用缓存），页面级样式根据自身的依赖引入不同的文件。

#### 问题

1. 哪些算是公共样式？
common文件夹下的所有样式文件？**common打包必须要有通用性，确保能够覆盖到大部分（甚至全部）页面，同时又不会太冗余，给页面带来很多不必要的样式**，这里必须要有一些规则来约束。

2. 打包的灵活性？
有些业务（活动等）就是单页面应用，打2个文件的方案真的适用吗？甚至H5的单页应用，就是需要直接inline到页面。

#### 一些尝试

1. 定义一个common.scss之类的文件，import一些项目级的css文件，页面link这个公共的样式文件。打包处理时，将这个打成一个包，页面其他的css全部打成另一个包。
2. common文件夹下存放的就是项目级的css文件，每个Page只关注页面级的样式，打包自动将common下的css打包，接入引入问题。
3. 灵活性可以通过配置的问题解决。
4. .......

### JS打包

JS打包方式多样性，fis 官方推荐的loader也没有很好的解决这个问题（PS：在入口函数的注释里写着：粗暴的打包方式）。

#### 来源

+ script 标签引入
+ require
+ require.async

#### 依赖方式

+ 同步
+ 异步

#### 处理方式

+ 同步直接script引入
+ 异步通过resourceMap加载

#### 奇葩

+ 同步里面的异步依赖
+ 异步里面的同步、异步依赖

以上只是问题的一小部分，足见打包场景的复杂和多样性，但是这里还是抛砖引玉的做一些简单的分析。

#### 提取来源

这里只提取require和require.async，不对script引入做任何处理。从loader的处理方式看，它对script的处理并不成功，需要添加各种ignore注释，偶尔会打乱script的顺序。

#### 打包逻辑

+ 分析每个文件id的依赖（深层分析，依赖的依赖）
+ 依赖和当前文件打包
+ script 插入到html中  （同步）
+ 生成resourceMap并插入到html中

#### 问题

1. 异\同步文件的异步依赖如何处理？
文件的异步依赖，全部打包到一起，丢到resourceMap中。
2. 公共库如何处理？
公共库单独打包，定义一个base之类的文件，将项目级的公共库引入，在html文件中require引入base，其他的文件的打包需要将这部分公共库排除掉。


### 其他问题

上面的逻辑也只是解决一些基础问题，鉴于**项目的多样性、处理方式的多样性**，实际只处理了一小部分的问题，很多例外的情况没有处理到：
+ require 第3方库
+ require.async 第3方库
+ require(模块是变量)
+ vm编译（vm中if else可以是2个页面，打包如何处理）
+ ......

fis 提供了packTo的打包方式，这个是最原始的，通过match正则打包成不同的文件，但是如何解决引入文件？打包的文件放到哪个html中？同步异步？

### 最后的最后

打包实际针对的是**一种业务场景和目录结构的方案，只是对一些默认的规则集的实现**，像packTo这种方式，貌似能够解决所有的打包场景（业务自己去配置），但这种蛮荒时代的方式真的可用吗？野蛮生长，没有内在的规律约束。

在打包之前，就应该**规定业务场景和目录结构，定义使用范围和边界**，不可能有一种打包方案使用所有的场景。官方的loader只是一种单页应用的方案。

