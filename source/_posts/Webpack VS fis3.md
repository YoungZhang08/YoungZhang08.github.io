---
layout: webpack
title: Webpack VS fis3
date: 2018-05-30 19:29:42
tags: 
- webpack
- fis3
categories: 前端构建工具
---

1. webpack 从 entry 出发，调用loader对模块的原始代码进行编译，接着调用acorn对js进行语法分析、收集其中的依赖关系，每个模块都会记录自己的依赖关系从而形成一颗关系树，最后调用compilation.seal进入render阶段根绝之前收集的依赖决定生成多少文件，每个文件的内容是什么。fis3 扫描项目目录拿到文件并初始化出一个文件对象列表，接着对文件对象中的每一个文件进行单文件编译，然后获取用户设置的package插件，进行打包处理。
<!--more-->
2. webpack 以js文件为入口，处理依赖的各种资源。fis3没有所谓的入口文件，所有文件（html，css，js等等）同等对待，自定义输出策略。
3. webpack 文件产出一般以entry文件为基准。fis3 任何文件可以发布到任何位置，相关引用的路径也自动更新。
4. webpack 适合类库打包和单页应用。fis3 适合多页应用的按需加载。
5. webpack 有hot reload功能，fis3内置的server目前不支持。