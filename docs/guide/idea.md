---
title: 基本概念
order: 2
group:
  path: /
nav:
  path: /guide
---

[form-render](https://x-render.gitee.io/form-render) 已经有了很多详细的一些基础的概念介绍和进阶教程，这里为了方便用户快速入门，
整理了目前自己觉得重要的一些基本概念出来扩展下文档 （其他一些详细的内容还是检验看官方文档）

一般动态表单都分为设计时，和运行时，用户在设计时配置表单的一些配置信息等，然后运行时就是表单实际的运行效果

## 设计时区域介绍

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/design.png)

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/componentDesign.png)

<Alert type="info">
设计时是基于fr-generator,组件除了上述一些常用的配置，还包括 defaultCommonSettings (配置这个字段，会给每一个自定义组件在区域5生成对应的自定义配置)
</Alert>

## 运行的真实效果

基于 form-render

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/run.png)

## Playground 通过配置 json 配置表单

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/playground.png)
