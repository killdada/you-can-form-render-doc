---
title: 数据联动
order: 4
group:
  title: 业务功能
  path: /business
nav:
  path: /guide
---

除了之前 [todo](/guide/intro#todo) 的第 5 点，针对以前的交互和业务，新增了数据联动处理，该功能类似以前的表单页面功能，整体功能交互上基本上和之前的没有差别。

## 简单联动

配置条件，满足条件之后，下方的配置的联动效果就会生效，比如如下图的功能，就是代表着: 如果产品类型单选按钮选择的值等于次票券，那么备注这个 textarea 的内容的值就是表达式的值 216；并且该 textarea 状态没有启用，表明无法进行二次编辑，是 disabled 禁用状态

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/datalink.png)

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/datalink1.png)

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/datalink2.png)

## 多选包含

存在组件是多选的时候，配置多选包含条件，如下图，如果多选条件，选择了武汉、杭州这二个选项 （只需要包含，也可以在选择其他选项，但是必须选择这二个才生效），那么下方联动备注禁用、非必填，且值为 67

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/datalink3.png)

## 表达式公式

输入框的值 等于 输入框 1 的值 `*` 9

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/datalink4.png)

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/datalink5.png)

## options 联动处理

有的时候我们需要满足某个条件的时候，对表单的一些 select 的 options 进行过滤，比如 产品类型单选按钮选择储值券的时候，对应的字段多选组件对应的固定 options 有原来的四个减少到了三个

![](http://qiniu.yenmysoft.com.cn/form-render-assets/business/datalink6.png)

## 提示

<Alert type="error">
1: 数据联动配置无法精确的检验里面的联动逻辑，可能存在联动1、联动2同时对一个字段进行更改，此时如果二个联动条件都满足，那么在后面的联动2会直接覆盖掉前面的联动1的处理

2：该联动交互大部分和之前的一致，会和 form-render 自带的联动处理冲突，因此最好不要同时配置这个联动、 form-render 自带的函数表达式联动
</Alert>
