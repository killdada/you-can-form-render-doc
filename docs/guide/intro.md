---
title: 介绍
order: 1
group:
  path: /
nav:
  title: 指南
  path: /guide
  order: 1
---

## 背景

1：现有的 erp 表单功能，交互体验差，性能也差

2：现有 erp 表单代码可维护性极差，文档缺失严重

在满足核心业务的前提下进行表单重构，重构注重以下三点

- 文档的完善性
- 用户的交互体验
- 代码的质量等

<Alert type="error">
  文档的完善性最为重要，一个好的文档可以降低很多维护成本
</Alert>

## 项目框架

基于 [umi3 脚手架](https://umijs.org/zh-CN/docs) 创建

基于 [x-render](https://x-render.gitee.io/form-render) 重构

## 业务梳理

梳理已有 erp 表单的业务功能点，逐步补充

### 核心需求

表单分为

- 申请表 跟流程关联
- 审批表 跟流程关联
- 功能表 单独使用

### 组件需求

之前支持的组件列表，后面需要一一验证

- [x] 输入框 (Input)
- [√] 选择器 (Select)
- [x] 文本 (Text)
- [x] 日期选择框 (DatePicker)
- [x] 上传 (Upload)
- [x] 单选框 (Radio)
- [x] 复选框 (Checkbox)
- [x] 按钮 (Button)
- [x] 表格 (Table)
- [x] 文本域 (Textarea)
- [x] 富文本(RTF)
- [x] 卡片 (Card)

## 概念

### form-render 概念

- [schema](https://x-render.gitee.io/form-render/schema/schema) schema 就是我们个性化配置的东西
- [rules](https://x-render.gitee.io/form-render/schema/rules) 检验相关
- [内置组件](https://x-render.gitee.io/form-render/schema/inner-widget)
- [自定义组件](https://x-render.gitee.io/form-render/advanced/widget)
- [其他用法](https://x-render.gitee.io/form-render)

### generator 配套表单设计器

- [fr-generator](https://x-render.gitee.io/tools/generator)

- [playload 体验](https://x-render.gitee.io/playground)

## 操作流程

```js
// 1 初始化依赖
yarn
// 2 开启本地调试
yarn dev
// 3 任意 id 进行表单设计
http://localhost:8000/#/design/2 (普通编辑)  http://localhost:8000/#/playground/2 （高级编辑） http://localhost:8000/#/design 新增
// 4 设计器对应的 id 访问运行时效果
http://localhost:8000/#/run/2
// 5 访问demo
http://localhost:8000/#/demo
```

[视频地址](http://qiniu.yenmysoft.com.cn/form-render-assets/videos/intro.mp4)

## todo

1.整体产品设计交互优化等

2.设计时，和运行时样式后续可能需要调整一轮

3.其他组件还没有实际验证过是否存在其他问题

4.根据我们的实际情况可能后续还需要增加适配其他自定义组件，包括 table 等，list 嵌套等

5.数据联动处理交互优化，和其他数据关联等优化

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/conditions.png)

运行时数据联动问题，虽然官方提供了

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/link.png)

这个图我们可以考虑配置一个 json 编辑器去编辑 schema 信息

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/link1.png)

内部监听值，部分常用的联动可以考虑封装到内部，比如省市区组件三级联动

但整体的联动相关性在设计时并不是非常友好，后续可能在设计器优化交互看怎么具体在界面配置上体现这个联动性

6.值的统一谁来处理，目前配置了菜单以后，可能会出现这个情况

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/rule.png)

这个是因为我们配置的时候，默认拖动组件以后都是携带类型 type 配置的，然后这二个字段的值来源于接口或者第三方数据，可能跟我们最开始默认组件配置的类型不对应导致检验错误

处理 1：在配置时手动配置更新该组件对应的类型

处理 2：在赋值的时候根据返回的值，动态更改对应组件 schema 的 type 类型

以及后续返回值以后提交到接口是否有限制条件？因为实际最后提交的数据都是严格跟 schema 组件定义的类型一一致提交的，接口是否已经兼容

7.目前绑定的第三方接口统一由前端发起请求，绑定的接口类型目前表单全局只支持绑定一个

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/form.png)

因为目前发现大部分页面组件绑定的数据来源都是来源于同一个表单，因此简单处理再表单全局绑定一次即可，
后续如果需要支持绑定多个接口，需要进行适配兼容优化交互

8.选择绑定本地数据表和远程第三方数据库表现上存在较大差异，整体返回的结构差异也比较大

本地

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/formlocal.png)

第三方

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/formthird.png)

但对应数据来说，这些都是属于远程数据，本地数据库应该属于第三方系统的一个子集，然后第三方系统也不应该有用户去输字段，
检验本地数据库直接合第三方系统融合在一起并且，选择绑定字段的时候都应该都需要支持下拉选择，这样用户配置的时候就不需要知道提前查看第三方系统对应的每一个的字段代表的含义，除了方便交互后续如果需要支持绑定多个也很方便处理

> 融合的时候，不清楚具体的接入第三方怎么获取某个接口所以的字段列表，可能需要额外维护

9.接口查询条件的未知性

![](http://qiniu.yenmysoft.com.cn/form-render-assets/home/queryCondition.png)

对于一个接口而言，如果有了接口文档一般就已经明确了该接口调用需要哪些参数，因此应该在选中某个接口的时候主动把需要的查询条件的主动填充上去，这样用户就不需要看具体的文档。

可能需要数据库额外维护这些数据，折中的简单的方式可以先在左侧 tips 提示下选中的接口对应的接口文档地址，这个也算是个优化交互点

10.绑定接口是否后续考虑支持 post 请求等

如果 query 存在过长导致无法正常发起请求的问题，此时可能考虑 post 请求的必要

11.其他之前的表单名称、表单类型、表单流程需要处理

## 项目目录结构

```js
form-render
├── app.config.js // build构建输出目录
├── commitlint.config.js // git-cz 格式
├── config
│   ├── config.dev.ts // 本地开发配置
│   ├── config.pre.ts // 预发布配置
│   ├── config.prod.ts // 生产配置
│   ├── config.test.ts // 测试环境配置
│   ├── config.ts // 入口配置
│   ├── routes.ts // 路由配置
│   └── sentry.ts // 添加sentry soucemap处理
├── deploy // 发布部署相关
│   ├── config
│   │   ├── base.json
│   │   ├── dev.json
│   │   ├── pre.json
│   │   └── prod.json
│   └── drone
│       └── dd-notification.tpl
├── dist // 生成的目录
│   ├── index.html
│   ├── umi.css
│   └── umi.js
├── mock // mock接口数据
│   ├── form.ts // 表单基础请求等
│   ├── json
│   │   ├── database.json // 本地数据库表的列表数据
│   │   ├── databaseDetailByName.json // 以表名为key存储的对应该表名的接口字段列表
│   │   ├── detail.json // 表单详情保存的业务数据
│   │   ├── linkpage.json // 之前的linkpage相关的数据，以第三方接口id或者以本地数据库表名为key
│   │   ├── schema.json // 以路由 /:id? 为key的 schema配置后的信息
│   │   ├── thirdDataBase.json // 第三方系统列表
│   │   ├── thirdDataBaseById.json // 以第三方系统id为key对应的接口列表
│   │   └── user.json // 用户信息数据
│   ├── link.ts // 表单数据绑定联动相关接口
│   ├── login.ts // 登录
│   └── util.ts // 读写json目录下的文件以模拟接口操作等
├── package.json
├── public
│   └── favicon.ico
├── README.md
├── scripts
│   └── rm-sourcemap.js // soucemap上传到sentry以后，删除构建后的生成的soucemap文件
├── src
│   ├── app.ts // app前置入口处理，在所有的渲染之前执行
│   ├── components // 公共组件
│   ├── constants // 公共变量
│   │   └── index.ts
│   ├── global.d.ts // 公共全局命名空间
│   ├── layouts // 页面入口布局
│   │   └── index.tsx
│   ├── less
│   │   └── global.less
│   ├── models // umi model
│   │   ├── formModel.ts // 远程数据绑定 （本地、第三方）的hook
│   │   └── schemaModel.ts // schema配置初始化，保存等hook
│   ├── pages
│   │   ├── document.ejs // index.html模板文件
│   │   └── Form // 表单相关页面
│   │       ├── Designer // 设计时入口
│   │       │   ├── index.tsx
│   │       │   └── settings // 个性化generator配置信息
│   │       │       ├── defaultCommonSettings.ts // 每一个组件注入可配置信息
│   │       │       └── defaultGlobalSettings.ts // 表单全局配置相关
│   │       │       └── defaultSettings.ts // 左侧组件列表配置
│   │       ├── hooks // 表单的hooks入口
│   │       │   └── index.ts
│   │       ├── Run
│   │       │   ├── index.less
│   │       │   └── index.tsx // 运行时入口
│   │       └── Widgets // 自定义组件
│   │           ├── Common // 设计运行时通用的自定义组件目录，一般比较少
│   │           ├── Designer // 设计时自定义组件目录
│   │           │     ├── components // 设计时自定义组件目录
│   │           │     │   ├── RemoteBind // 远程数据绑定 （form、select都用的这个文件）
│   │           │     │   │   ├── FixedOptions // select组件绑定固定options的处理
│   │           │     │   │   ├── QueryConditions // 绑定远程接口的时候需要处理的查询条件
│   │           │     │   │   ├── utils.ts // 辅助工具函数
│   │           │     │   │   └── index.tsx // 组件入口
│   │           │     │   ├── Data // 数据绑定
│   │           │     │   ├── Form // 表单数据绑定
│   │           │     │   ├── Select // 单选、多选数据绑定
│   │           │     │   └── index.tsx
│   │           │     ├── Data // 数据绑定
│   │           │     ├── Form // 表单数据绑定
│   │           │     ├── Select // 单选、多选数据绑定
│   │           │     └── index.tsx
│   │           ├── index.ts
│   │           └── Run // 运行时自定义组件目录
│   ├── service
│   │   ├── ApiConfig.ts // 所以接口地址
│   │   ├── FormService.ts // 表单相关接口导出
│   │   ├── index.ts
│   │   └── LoginService.ts // 登录相关接口
│   ├── types
│   │   ├── form.ts // 表单相关类型接口
│   │   ├── index.ts
│   │   ├── schema.ts // 描述schema结构
│   │   └── service // service 层（目录下）接口返回的类型结构
│   │       ├── form.ts
│   │       ├── index.ts
│   │       └── login.ts
│   └── utils // 工具函数
│       ├── form // 表单工具函数
│       │   ├── check.ts // 检测设计时提交的id是否重复 （后续可以考虑去掉，现在id不允许编辑）
│       │   ├── getDataBind.ts // 处理
│       │   ├── index.ts
│       │   └── tool.ts // form-render配置的设计器generator里面的 src/utils 方法迁移过来
│       ├── index.ts
│       ├── reg.ts // 通用正则
│       ├── request.ts // 请求接口封装
│       ├── sentry.ts // sentry初始化
│       ├── systemVar.ts // 系统变量处理 （之前的session和缓存）
│       └── user.ts // 存储用户信息
├── tree.md
├── tree.text
├── tsconfig.json
├── typings.d.ts
└── yarn.lock

```

> src/utils/form/tool 里面收集了第三方 generator utils 部分方法，有些方法对我们很有用，我们可以多看看 x-render 里面源码的部分实现加深体验

## 代码规范约束

### 目录文件命名约束

1.1 所有的 types service 定义完目录下的文件以后，统一在目录下定义 index.ts 统一抛出对应目录下所有的文件方法,方便其他地方引入和管理维护

```js
src / Widgets / index.ts;

// 自定义组件-设计时使用
import Data from './Data';
import Form from './Form';
import Select from './Select';

export default {
  mdData: Data,
  mdForm: Form,
  mdSelect: Select,
};
```

1.2 如果是组件，目录统一使用大写，hook 统一 use 开头

1.3 src/models (umi model) 下的文件统一 以 Model 结尾 如 formModel.ts

1.4 src/pages/Form 下严格区分 Designer Run Widgets

1.5 src/pages/Form/Widgets 严格区分 Designer Common Run

1.6 src/service 下目录文件严格以 Service 结尾 如 FormService

1.7 src/service 下定义了 FormService 同步在 src/types/service 下定义以同名前缀定义的 form.ts 接口类型

### 业务逻辑命名约束

2.1 定义的表单自定义组件 widgets 统一在 src/types/form.ts 下增加枚举

```js
/**
 * @description 支持的自定义组件类型 md前缀代表设计时使用，mr前缀运行时使用，mz前缀设计运行通用
 * @export
 * @enum CustomComponentsWidgets
 */
export enum CustomWidgetsTypes {
  /**
   * @description 数据绑定组件，设计时使用
   */
  mdData = 'mdData',
  /**
   * @description 表单绑定接口组件，设计时使用
   */
  mdForm = 'mdForm',
  /**
   * @description radio、checkbox、select等组件绑定数据来源，设计时使用
   */
  mdSelect = 'mdSelect',
}
```

> 特别注意其中组件 md 前缀代表设计时 widgets, mr 代表运行时，mz 代表设计运行时通用，这样定义配置的时候更加明确需要渲染的是哪个组价

2.2 src/types 里面的除了 service 目录下的类型（接口返回），其他文件类型一律记得给每个属性添加注释， 后续考虑考虑健全 service 和接口，通过 protocol 描述，通过工具生成文档和 ts 类型，现在都是借助于 vscode Json to TS 插件生成的

2.3 尽量不要用 any，除非是一些特殊情况返回值不确定的时候，其他使用第三方库的时候方法的时候可以根据提示增加一些类型供我们使用

2.4 定义 form-render 自定义组件时 props 继承 global.d.ts,方便用户知道先有自定义组件接受的 props

```js
// src/pages/Form/Widgets
import type { FC } from 'react';
import React from 'react';

import type { SelectBindData } from '@/types';
import { RemoteBind } from '../components';

// 大部分情况下发现，先有的字段数据绑定都是绑定到同一个接口地址，很少绑定多个地址，(这里暂时忽略绑定多个接口地址）
const SelectBind: FC<Generator.CustomComponentsProp<SelectBindData>> = (
  props,
) => {
  // 别忘记绑定key，没有绑定key，切换不同的组件进行配置，select弹窗内容可能还保留上一次的 ($id必填并且保存的时候检验下唯一性,嵌套结构id可能后续允许重复，这里后续等有多层对象结构再优化) 或者所有都共用同一个实列，但是初始化的时候注意处理赋值，不然可能打开弹窗的时候填充的是上一次的值
  return (
    <RemoteBind
      key={props.addons.formData.$id}
      {...props}
      type="select"
    ></RemoteBind>
  );
};

export default SelectBind;

// src/global.d.ts
/**
 * @export
 * @interface CustomComProp
 * @description 自定义组件接受的props描述
 * @description 该类型定义来源于(https://x-render.gitee.io/form-render/advanced/widget#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6%E6%94%B6%E5%88%B0%E7%9A%84-props)
 * T value 的结构
 */
export interface CustomComponentsProp<T> {
  /**
   * @description  是否禁止输入
   */
  disabled: boolean;
  /**
   * @description  是否只读
   */
  readOnly: boolean;
  /**
   * @description  组件现在的值
   */
  value: T;
  /**
   * @description  函数，接收 value 为入参，用于将自定义组件的返回值同步给 Form
   */
  onChange: (value: any) => void;
  /**
   * @description  组件对应的子 schema
   */
  schema: Record<string, unknown>;
  /**
   * @description  其他对应的级联属性
   */
  addons: {
    /**
     * @description  注意挂在 addons 下面。用于在本组件内修改其他组件的值 onItemChange(path, value)
     */
    onItemChange: (path: string, value: any) => void,
    /**
     * @description   getValue 获取表单当前的数据
     */
    getValue: (path?: string) => any,
    /**
     * @description   表单当前的数据。其实可以通过 getValue 获取，但我也传下来了。
     */
    formData: any,
    /**
     * @description   目前数据所在的 path，例如"a.b[2].c[0].d"，string 类型。
     */
    dataPath: string,
    /**
     * @description  如果 dataPath 不包含数组，则为 [], 如果 dataPath 包含数组，例如"a.b[2].c[0].d"，则为 [2,0]。是自上到下所有经过的数组的 index 按顺序存放的一个数组类型
     */
    dataIndex: number[],
  };
  /** @description select等绑定options时自定义组件对应的类型，对应内置组件四个，暂时保留，后续如果四组件有特殊要求基于该字段扩展补充 */
  typePorps?: string;
  /**
   * @description select等相关组件，需要区分单选和多选
   */
  comType: 'sinele' | 'multiple';
  /**
   * @description 预留字段
   */
  [argName: string]: unknown;
}
```

2.5 定义 ant-design 自定义渲染 FormItem 继承基类 src/types/form.ts

```js
/**
 * @export
 * @interface CustomFormItemProps
 * @template T
 * @description antd 自定义formitem包裹需要的结构
 */
export interface CustomFormItemProps<T> {
  /**
   * 组件接受到的props值
   */
  value?: T;
  /**
   * 改变数据提交到父组件
   */
  onChange?: (value: T) => void;
  /**
   * 只是为了去掉编辑器的提示，比如该组件需要ref 提供方法给父组件使用的时候
   */
  ref?: any;
}
```

## 参考文档

- [umi 框架](https://umijs.org/zh-CN/docs)
- [ant-design 组件库](https://ant.design/docs/react/introduce-cn)
- [ahook 通用 react hook](https://ahooks.js.org/zh-CN)
- [procomponents ant form table 等组件二级封装](https://procomponents.ant.design/)
- [form-render 表单渲染方案](https://x-render.gitee.io/form-render)
- [genertor 配套表单设计器](https://x-render.gitee.io/tools/generator)
- [@hyx/utils 工具库](http://mz-common-doc.maizuo.com/tools/utils/common)

> ahooks 部分 hook 可能有你意想不到的功能，比如 useRequest 可以处理很多情况，建议过一些拥有的 api，到实际场景需要的时候可以从里面选用 hook

## 参考交互文档

- [Avue](https://form.nutflow.vip/)
- [fRender](https://dream2023.gitee.io/f-render/)
- [designable](https://designable.netlify.app/)
- [form-create](http://form-create.com/designer/?fr=github)
- [vue-form-design](https://jjxliu306.gitee.io/vue-form-design/)
- [k-form-design](http://cdn.kcz66.com/k-form-design.html)

> 现有的基础版本有很多功能都需要进一步完善添加，交互方面可以参考其他动态表单方案的的体验

## 提示

<Alert type="error">

上述指南，比如 todo 已经解决了一部分，和项目目录结构新增了很多其他文件，但是整体思路和目录接口等都没有发生大的改变这里就不再统一更新，后续主要关注

1： 从业务功能的角度上来分别介绍以前的业务功能如何在新版本操作实现

2： 代码层面介绍各个业务自定义组件的实现逻辑和注意事项

3： 提供 FAQ 方便快速定位查找一些常用问题

</Alert>
