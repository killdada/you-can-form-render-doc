---
nav:
  title: Schema
  path: /schema
  order: 1
---

# Schema

## Schema 基础描述结构

schema 配置信息字段说明等， 基于 [schema 介绍](https://x-render.gitee.io/form-render/schema/schema) 扩展新增属于自己项目的接口说明

所有的类型来源地址 [form 类型](https://github.com/killdada/you-can-form-render/blob/master/src/types/form)

```js
// Schema结构很灵活，这里定义一些基础常见的结构即可
import type { FormTypes } from './base';
import type { FormBindData, IDataBind, RemoteBindData } from './dataBind';
import type { DataLinkItem } from './dataLink';
import type {
  ETableConfig,
  ETableSort,
  TableSettingDataSourceType,
} from './table';

// 上面这些基本是内置的schema结构类型
export interface IBaseSchemaCommonProps {
  /** @description 组件唯一id 字段名称/英文 */
  $id: string;
  /** @description 标题 */
  title: string;
  /** @description 说明 */
  description: string;
  /** @description 默认值 */
  default: any;
  /** @description 必填 */
  required: boolean;
  /** @description 占位符 */
  placeholder: string;
  /** @description 数据类型 */
  type: 'string' | 'number' | 'boolean' | 'array' | 'object' | 'range' | 'html';
  /** @description 最小值 */
  min: number;
  /** @description 最大值 */
  max: number;
  /** @description 禁用 */
  disabled: boolean;
  /** @description 只读 */
  readOnly: boolean;
  /** @description 隐藏 */
  hidden: boolean;
  /** @description 元素宽度 */
  width: number;
  /** @description 标签宽度 */
  labelWidth: number;
  /** @description 检验规则 */
  rules: any[];
  /** @description 组件内置props */
  props: any;
  /** @description options对应的value */
  enum: (string | number)[];
  /** @description options对应的label */
  enumNames: (string | number)[];
  /** @description 指定要那个组件渲染 */
  widget: string;
  [key: string]: any;
}

// 根据业务实际情况扩展的类型
export interface ISchemaCommonProps extends Partial<IBaseSchemaCommonProps> {
  /** @description 数据绑定 */
  dataBind: IDataBind;
  /** @description options类数据绑定 */
  enumDataBind: RemoteBindData;
  /** @description 表格列配置绑定 */
  columnTableDataBind: TableSettingDataSourceType[];
  /** @description 表格是否可以导入 */
  canImport: boolean;
  /** @description 表格是否可以导出 */
  canExport: boolean;
  isAddOrDel: ETableConfig[];
  /** @description 表格新增的排序方式 */
  addType: ETableSort;
  [key: string]: any;
}

// 自带的基础结构
export interface IBaseSchema {
  /* @description 结构类型 */
  type: string;
  /* @description 上下排列，还是左右排列 */
  displayType: 'column' | 'row';
  /* @description 一行展示多少个 */
  column: number;
  /* @description 全局标签宽度 */
  labelWidth: number;
  /* @description schema object对应需要的properties */
  properties: Record<string, ISchemaCommonPropsPartial>;
  [key: string]: any;
}

export interface ISchema extends Partial<IBaseSchema> {
  /* @description 表单远程数据全局绑定 */
  formDataBind: FormBindData;
  /* @description 表单联动数据绑定 */
  dataLink: DataLinkItem[];
  formName: number;
  /* @description 表单的类型 */
  formType: FormTypes;
  /* @description 表单所属分类 */
  formCategory: number;
  /* @description 审批表单管理的表单id */
  relFormId: string | number;
  /* @description 表单描述 */
  formDesc: number;
  /* @description 审批表是否需要审批 */
  isNeedApprove: boolean;
  [key: string]: any;
}

export type ISchemaPartial = Partial<ISchema>;
export type ISchemaCommonPropsPartial = Partial<ISchemaCommonProps>;
```

<Alert type="info">
基于业务情况扩展了一些字段，几个重要的扩展接口字段说明如下
</Alert>

## FormBindData 表单数据绑定

表单在设计时配置配置远程接口绑定

一般情况下，现有业务里面很多字段的数据绑定都是直接来源于同一个接口，以前的方式需要一一对每一个字段单独设置绑定，这里优化成了支持在设计时直接配置全局的唯一的远程接口地址用户绑定字段直接选择组件进行绑定即可

```js

export enum ERemoteBindType {
  /**
   * 根据数据库来源表
   */
  database = 'database',
  /**
   * 根据第三方系统来源表
   */
  thirdDataBase = 'thirdDataBase',
  /**
   * 固定值，select 等需要 options 采取本地输入的形式
   */
  fixed = 'fixed',
}

/**
 * @description 查询条件参数的来源方式枚举
 */
export enum QueryConditionsValMethod {
  /**
   * @description 固定值
   */
  fixed = 'fixed',
  /**
   * @description 来源于url参数
   */
  url = 'url',
  /**
   * @description 来源于本页面字段
   */
  field = 'field',
  /**
   * 来源于系统变量
   */
  system = 'system',
}

/**
 * @description 查询条件参数的值，支持string,number类型
 */
export type QueryConditionsParamType = 'string' | 'number';

/**
 * @description 接口查询条件
 * @export
 * @interface QueryConditionsVal
 */
export interface QueryConditionsVal {
  /**
   * @type {string}
   * @description 当前行 id
   */
  id: React.Key;
  /**
   * @description 参数名，英文
   */
  name?: string;
  /**
   * @description 查询条件参数的值，支持string,number类型
   */
  type?: QueryConditionsParamType;
  /**
   * @description 查询条件参数的值来源方式，枚举自 QueryConditionsValMethod
   */
  method?: QueryConditionsValMethod;
  /**
   * @description 查询条件的值或者key
   */
  value?: string;
  /**
   * @description 该参数是否必须
   */
  required?: boolean;
}


export interface FormBindData {
  /**
   * @description 数据来源方式
   */
  dataSourceMethod: ERemoteBindType | undefined;
  /**
   * @description 数据来源表名
   */
  databaseTableName?: string;
  /**
   * @description 第三方系统 Id
   */
  appId?: string | number;
  /**
   * @description 第三方系统 接口名称 ID
   */
  appInterId?: string;
  /**
   * @description 第三方系统 接口相对地址
   */
  appInterAddr?: string;
  /**
   * @description 接口查询条件
   */
  queryConditions: QueryConditionsVal[];
  /**
   *  @description 其他字段，组装数据的时候可能会额外增加一些字段
   */
  [argName: string]: any;
}

```

## IDataBind 组件数据绑定

通过 FormBindData 全局绑定了接口以后，可以通过这个组件给组件绑定远程字段 key，也可以选择绑定其他远程接口，或者直接绑定系统变量

```js

/**
 * @description 数据绑定组件取值方式的枚举
 */
export enum EDataBindType {
  /**
   * 根据远程获取数据，表单配置里面表单数据绑定来源选择 database  thirdDataBase 该值为真
   */
  remote = 'remote',
  /**
   * remote是直接绑定表单配置里面的接口对应的字段，otherRemote允许用户直接跟接口进行关联 （同一个表单里面的字段数据绑定接口支持多个接口）
   */
  otherRemote = 'otherRemote',
  /**
   * 系统变量
   */
  system = 'system',
}

/**
 * @description 单选多选等选项接口
 */
export interface IOptions {
  /** @description react key */
  id: React.Key;
  /** @description label显示名称 */
  label: string | number;
  /** @description 选项值 */
  value: string | number;
  /** @description 选项是否默认 */
  default: boolean;
}

/**
 * @description select等需要绑定数据来源的配置结构
 */
export interface SelectBindData extends FormBindData {
  /**
   * @description 固定options列表数据
   */
  options?: IOptions[];
  /** @description 远程 options 取的label 数据库字段key */
  label?: string | number;
  /** @description 远程 options 取的value 数据库字段key */
  value?: string | number;
}

export type RemoteBindData = SelectBindData;

/**
 * @export
 * @interface IDataBind
 * @description 数据绑定组件接受的props.value值
 */
export interface IDataBind extends Partial<RemoteBindData> {
  /**
   * @description 数据绑定方式
   */
  sourceMethod?: EDataBindType;
  /**
   * @description 数组绑定的字段
   */
  field?: string | TableDataBindSetting[];
}
```

## TableSettingDataSourceType table 组件数据绑定

table 组件列配置信息等

```js

/**
 * @export
 * @enum {number}
 * @description 列配置每一列渲染的组件类型枚举
 */
export enum TableColumnComType {
  /** @description 纯文本 */
  text = 0,
  /** @description 输入框 */
  input = 1,
  /** @description 按钮 */
  button = 2,
  /** @description 日期选择器 */
  datepick = 3,
  /** @description 选择器 */
  select = 4,
  /** @description 时间选择器 */
  timepick = 5,
}

/**
 * table组件 列
 */
export interface TableSettingDataSourceType {
  /** @description 列组件类型 */
  rowType?: TableColumnComType;
  /** @description 列名称 */
  rowTitle?: string;
  /** @description 是否显示 */
  isShow?: boolean;
  /** @description 是否金额字段 */
  isMoney?: boolean;
  /** @description 是否必填 */
  isRequired?: boolean;
  /** @description 是否禁用 */
  isDisabled?: boolean;
  /** @description 当时select选项的时候是否支持多选 */
  isMultiple?: boolean;
  /** @description 列id */
  id: React.Key;
  /** @description 当rowType为select组件时select绑定的数据 */
  optionState?: SelectBindData;
  /** 对应数据绑定的字段信息key */
  fieldKey?: string;
}

/** @description 数据绑定的时候如果是table组件只需要知道这几个值即可 */
export type TableDataBindSetting = Pick<
  TableSettingDataSourceType,
  'id' | 'rowTitle' | 'fieldKey' | 'rowType',
>;
```

## DataLinkItem 数据联动配置信息

数据联动规则配置等

```js
/** 数学符号枚举 */
export type MathSign = '===' | '!==' | '>' | '>=' | '<' | '<=' | 'includes';
export type MathSignText = '等于' | '不等于' | '大于' | '大于等于' | '小于' | '小于等于' | '包含';
export enum EMathSign {
  '===' = '等于',
  '!==' = '不等于',
  '>' = '大于',
  '>=' = '大于等于',
  '<' = '小于',
  '<=' = '小于等于',
  'includes' = '包含',
}

/**
 * @description 数据联动先决条件
 */
export interface DataLinkCondition {
  /** react key */
  id: React.Key;
  /** 字段名称key */
  field: string;
  /** 数学符号 */
  sign: MathSign;
  /** 值类型 */
  type: 'string' | 'number' | 'boolean';
  /** 值,点击、下拉多选可能是数组 */
  value: string | (string | number)[];
  /** 条件组合方式 */
  combine: 'and' | 'or';
}

export interface IFormula {
  /** 当前编辑列的id */
  id?: React.Key;
  /** 展示的表达式，字段label拼接而成 */
  label: string;
  /** 真正的表达式，字段value（id）拼接而成 */
  value: string;
}

/**
 * @description 数据联动根据条件设置的联动table列表
 */
export interface DataLinkAction {
  /** react key */
  id: React.Key;
  /** 字段名称key */
  field: string;
  /** 状态,是否启用 */
  isEnable: boolean;
  /** 可见性 */
  isVisible: boolean;
  /** 必填性 */
  isRequired: boolean;
  /** 不满足是是否清空 */
  isClear: boolean;
  /** 值 */
  formula?: IFormula;
  /** 从已有的select选项勾选出来需要展示的options */
  options?: (string | number)[];
  /** formula里面的公式计算后的值 */
  value?: string | number | undefined;
  /** 联动actions动作依赖的条件字段id列表 */
  conditionsFieldIds?: (string | number)[];
}

/**
 * @description 数据联动列表项
 * @interface DataLinkItem
 */
export interface DataLinkItem {
  /** 联动名称 */
  name: string;
  /** 联动id */
  id: React.Key;
  /** 是否启用 */
  isEnable: boolean;
  /** 数据联动先决条件 */
  conditions?: DataLinkCondition[];
  /**  数据联动根据条件设置的联动table列表 */
  actions?: DataLinkAction[];
  /** 备注 */
  desc?: string;
}
```
