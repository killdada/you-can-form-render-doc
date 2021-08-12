---
title: 数据绑定
order: 1
group:
  title: 业务功能
  path: /business
nav:
  path: /guide
---

## 全局表单唯一绑定接口

针对之前的 erp 表单数据绑定大部分表单都是绑定的同一个接口数据来源，因此支持在表单配置里面进行全局管理

1：首先我们需要绑定远程的接口

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/databind.png)

2: 然后给需要的组件绑定字段

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/databind1.png)

## 其他自定义接口绑定

以前的 erp 表单也存在绑定多个接口的情况，因此也支持了一个表单的数据来源于多个接口

直接选中需要的组件，给组件绑定远程接口和字段

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/databind2.png)

## 系统变量

对照以前的系统变量绑定处理

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/system.png)

> 后续考虑，直接在这里面进行扩展，新增以前系统的缓存功能数据绑定
