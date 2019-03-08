---
title: 基于vue的脚手架开发与发布到npm仓库
date: 2019-01-17 16:01:58
tags: cli, vue, node
cover: 'https://flaviocopes.com/nodejs/banner.png'
---
### 什么是脚手架
在项目比较多而且杂的环境下，有时候我们想统一一下各个项目技术栈或者一些插件/组件的封装习惯。但是每次从零开发一个新项目的时候，总是会重复做一些类似于复制粘贴的工作是一个很头疼的事情，所以各种各样的脚手架应用而生，
脚手架也就是为了方便我们做一些重复的事情，快速搭建一个基本的完整的项目结构。例如：vue-cli, react-cli, express-generator

### 构建一个简单的bin文件
package.json 里面的bin字段，是声明一个命令名和本地文件名的映射，在安装时，如果是全局安装，npm会使用符号链接将这些文件链接到prefix/bin,如果是本地安装，会链接到项目内node_modules下的bin目录下。这样，我们
在下载包后，就可以使用包内部bin声明的对应命令了。这里以vue-cli为例：

1. 全局安装vue-cli
```
npm install vue-cli -g
```
2. 找到脚手架的位置，这里是全局安装的，会定位到 /usr/local/lib/node_modules/vue-cli

### 具体交互逻辑和命令补充

### 发布到npm

### 测试