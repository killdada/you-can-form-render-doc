---
title: 数据填充、联动
order: 2
group:
  title: 功能实现
  path: /business
---

## 数据填充

运行时根据设计配置的远程数据进行数据填充处理

主要代码入口

### [远程接口整理](https://github.com/killdada/you-can-form-render/blob/master/src/utils/form/getDataBind.ts)

```js
import { getQuery } from '@hyx/mz-utils';
import type {
  SelectBindData,
  IDataBind,
  FormBindData,
  QueryConditionsVal,
  BindComponentType,
  RemoteBindData,
  IFormatRemoteData,
} from '@/types';
import { CustomWidgetsTypes } from '@/types';
import { EDataBindType, ERemoteBindType, QueryConditionsValMethod } from '@/types';

import { flattenSchema, getKeyFromUniqueId } from './tool';
import { getSystemData } from '../systemVar';
import { optimizeRemoteData } from './remoteData';

/**
 * @description 根据schema配置的查询条件组装返回需要的查询参数
 */
function getQueryCondition(conditions: QueryConditionsVal[] = []) {
  const res: Record<string, any> = {};
  // eslint-disable-next-line no-restricted-syntax
  for (const item of conditions) {
    const { value = '', method, name = '', type, required } = item;

    // 从url取数
    if (method === QueryConditionsValMethod.url) {
      const val = getQuery(value);
      // 必需参数但是没有取到值，直接抛出错误
      if (required && !val.length) {
        throw new Error(`查询条件${name}是必要条件（URL）！`);
      }
      res[name] = type === 'string' ? val : parseInt(val, 10);
    } else if (method === QueryConditionsValMethod.fixed) {
      // 固定值在提交的时候已经检验了不能为空，不需要判断required
      res[name] = type === 'string' ? value : parseInt(value, 10);
    } else if (method === QueryConditionsValMethod.system) {
      // 系统变量直接从 src/app.ts 入口文件获取的用户信息里面获取，暂不处理系统变量包含之前缓存的功能
      const val = getSystemData(value);
      // 必需参数但是没有取到值，直接抛出错误
      if (required && typeof val === undefined) {
        throw new Error(`查询条件${name}是必要条件（系统变量）！`);
      }
      res[name] = type === 'string' ? val : parseInt(val, 10);
    }
    // 本页面字段可能需要考虑更多东西，暂不处理
  }
  return res;
}

/**
 * @description 获取远程数据时需要的接口参数
 */
export function getApiQuery({
  data,
  id,
  type,
}: {
  data: Partial<RemoteBindData>;
  id: string;
  type: BindComponentType;
}): IFormatRemoteData {
  let query: Partial<RemoteBindData> = {};
  let result: IFormatRemoteData = { id, type };

  if (data.dataSourceMethod === ERemoteBindType.database) {
    // 数据库表名带过去
    query.databaseTableName = data.databaseTableName;
  } else {
    query.appId = data.appId;
    query.appInterId = data.appInterId;
  }
  query = { ...query, ...getQueryCondition(data.queryConditions) };

  if (type === 'select') {
    // 额外传递label value，请求完select接口以后根据这个设置对应的label，value
    result = {
      ...result,
      label: data.label,
      value: data.value,
    };
  }
  return { ...result, query };
}

/**
 *
 *
 * @description 根据接口返回的schema组装处理跟数据绑定相关的数据 dataBind （数据绑定）formDataBind 表单数据绑定 enumDataBind 下拉等数据绑定
 * @param {*} schema 接口获取的schema信息
 * @return {*}  {{
 *   remoteDatas: IFormatRemoteData[]; 远程绑定数据集合
 *   bindData: Record<string, any>; 本地数据绑定的系统变量
 * }}
 */
export function getDataBind(schema: any): {
  remoteDatas: IFormatRemoteData[];
  bindData: Record<string, any>;
} {
  const flattenData = flattenSchema(schema);
  // form里面没想到实列已经返回了扁平的数据
  // console.log('flattenData', flattenData);
  const remoteDatas = <any>[];
  const bindData: Record<string, any> = {};
  Object.entries(flattenData).forEach(([$id, item]) => {
    const {
      enumDataBind = {},
      formDataBind = {},
      dataBind = {},
      widget = '',
    }: {
      enumDataBind: Partial<SelectBindData>;
      formDataBind: Partial<FormBindData>;
      dataBind: Partial<IDataBind>;
      widget?: string;
    } = item.schema || {};

    const isTable = widget === CustomWidgetsTypes.mrTable;

    // 当前组件对应的id，现在只考虑一级的情况可以直接用这个设置值
    const id = getKeyFromUniqueId($id);

    if (enumDataBind.dataSourceMethod && enumDataBind.dataSourceMethod !== ERemoteBindType.fixed) {
      remoteDatas.push(getApiQuery({ data: enumDataBind, id, type: 'select' }));
    }

    if (formDataBind.dataSourceMethod) {
      // form全局配置接口返回的字段应该是页面管理该接口的所有的key，这里暂不处理全量返回赋值
      remoteDatas.push(getApiQuery({ data: formDataBind, id, type: 'form' }));
    }

    // 现在表格数据在组件里面自行处理，不在全局处理 （可以考虑在这里处理表格的数据，）
    if (dataBind.sourceMethod) {
      if (dataBind.sourceMethod === EDataBindType.remote) {
        // 远程的话，本地的已经绑定字段并且更新到bind里，第三方的也是更新到bind里，后续直接用表单详情接口赋值即可，测试不通过的话再额外处理，统一在formDataBind里面已经处理了这种情况，只需要请求下来全局表单接口即可·1··
      }
      // 处理系统变量, 表格系统变量被禁用，表格的field是数组
      if (dataBind.sourceMethod === EDataBindType.system && !isTable) {
        const val = getSystemData((dataBind.field || '') as string);
        bindData[id] = val;
      }

      // 其他字段单独绑定第三方远程接口
      if (dataBind.sourceMethod === EDataBindType.otherRemote) {
        remoteDatas.push({
          ...getApiQuery({ data: dataBind, id, type: 'data' }),
          field: isTable ? id : dataBind.field || '',
          isTable,
        });
      }
    }
  });

  const remote = optimizeRemoteData(remoteDatas);

  return {
    remoteDatas: remote,
    bindData,
  };
}
```

整体思路，遍历 schema 每个组件对应的配置信息，分别把配置了远程接口绑定的一一存储到 remoteDatas， 比如 select 远程选项绑定、data 数据绑定多个接口等，其他系统缓存变量直接通过 `getSystemData` 获取到对应的值然后填充到对应的 key 接口, `getApiQuery`是用来处理远程接口绑定的查询条件组装优化的

### [远程接口合并优化](https://github.com/killdada/you-can-form-render/blob/master/src/utils/form/remoteData.ts)

第一步通过字段配置信息，获取远程接口对应的配置信息等，存在多个组件绑定同一个远程接口的情况，这个时候就需要把相同的接口直接过滤掉，避免发起多个重复请求

```js
import type { IFormatRemoteData } from '@/types';

// [
//   { id: 'input_8grVsh', type: 'data', query: { appId: 23, appInterId: 190 } },
//   { id: 'input_8grVsh_jmXqty', type: 'data', query: { appId: 23, appInterId: 190 } },
//   {
//     id: 'select_hd1W3s',
//     type: 'select',
//     label: 'categoryName',
//     value: 'categoryId',
//     query: { appId: 2, appInterId: 122 },
//   },
//   {
//     id: 'select_hd1W3s_DwvLw-',
//     type: 'select',
//     label: 'categoryName',
//     value: 'categoryId',
//     query: { appId: 2, appInterId: 122 },
//   },
//   {
//     id: 'select_hd1W3s_DwvLw-_lKzMWI',
//     type: 'select',
//     label: 'categoryName1',
//     value: 'categoryId',
//     query: { appId: 2, appInterId: 122 },
//   },
//   { id: '#', type: 'form', query: { appId: 2, appInterId: 30 } },
//   { id: 'table_VAqQK5', type: 'data', query: { appId: 23, appInterId: 190 } },
// ];

// 转化后的请求数据结构
// [
//   { id: '#', type: 'form', query: { appId: 2, appInterId: 30 } },
//   {
//     id: 'input_8grVsh',
//     type: 'data',
//     query: { appId: 23, appInterId: 190 },
//     field: ['customer1', 'customer'],
//     isTable: false,
//   },
//   {
//     id: 'select_hd1W3s',
//     type: 'select',
//     label: 'categoryName',
//     value: 'categoryId',
//     query: { appId: 2, appInterId: 122 },
//     selectData: [
//       { id: 'select_hd1W3s', label: 'categoryName', value: 'categoryId' },
//       { id: 'select_hd1W3s_DwvLw-', label: 'categoryName', value: 'categoryId' },
//       { id: 'select_hd1W3s_DwvLw-_lKzMWI', label: 'categoryName1', value: 'categoryId' },
//     ],
//   },
//   {
//     id: 'table_VAqQK5',
//     type: 'data',
//     query: { appId: 23, appInterId: 191 },
//     field: ['table_VAqQK5'],
//     isTable: true,
//   },
// ];

// 当页面绑定数据接口如上结构，其中数组1数组2绑定的是同一个接口，数组3数组4也是同一个接口，数组5和3,4是同一个接口但是绑定的labbel不同，这里做一层优化处理，过滤掉相同的接口地址 （接口地址一样，参数一样，现在的情况主要就是query一样）

export function optimizeRemoteData(data: IFormatRemoteData[] = []): IFormatRemoteData[] {
  const result: IFormatRemoteData[] = [];
  const idsMap = {} as any;
  data.forEach((item) => {
    const { type, query, isTable = false, ...other } = item || {};
    // 全局表单只有一个绑定不存在重复
    if (type === 'form') {
      result.push(item);
    } else if (type === 'select' || type === 'data') {
      // 用type ，query当唯一key，现在只有比较这二个就知道是不是重复的接口，isTable 也需要加进来区分
      const key = `${isTable}${type}${JSON.stringify(query)}`;
      // 存在，值拼接
      if (idsMap[key]) {
        if (type === 'select') {
          idsMap[key].selectData.push(other);
        } else {
          idsMap[key].field.push(item.field);
        }
      } else if (type === 'select') {
        // 不存在先初始化
        idsMap[key] = {
          ...item,
          selectData: [other],
        };
      } else {
        idsMap[key] = {
          ...item,
          field: [item.field],
        };
      }
    }
  });
  Object.keys(idsMap).forEach((key: string) => {
    result.push(idsMap[key]);
  });
  return result;
}

```

### [运行时入口](https://github.com/killdada/you-can-form-render/blob/master/src/pages/Form/Run/index.tsx)

```js
const fetchRemoteData = useCallback(async () => {
  setTrue();
  const { remoteDatas, bindData } = getDataBind(schema);
  // 远程接口数据
  const promises = remoteDatas.map((item) => FormService.fetchRemoteFormData(item.query));
  const remoteDataResult = await Promise.all(promises);
  // 本身表单详情业务数据,以前业务已经数据整合到design同一个接口了
  // const { data: detail } = await FormService.fetchFormDetail(id);
  const detail = formBusinessData.businessData || {};
  // 本地系统变量的默认值等
  let values = { ...bindData };
  remoteDataResult.forEach((item, index: number) => {
    const data = item.data as any;
    const remoteData = remoteDatas[index] || {};
    const { type, selectData = [], field = [], isTable = false } = remoteData;
    if (type === 'form') {
      // 表单配置全局绑定接口，然后组件绑定这个接口对应的字段，需要把改接口数据最后拼接到值里面去 (后面考虑过滤下多余字段)
      values = { ...values, ...data };
    } else if (type === 'select') {
      selectData.forEach((selectItem) => {
        const { value = '', label = '' } = selectItem;
        form.setSchemaByPath(selectItem.id, {
          enum: data.map((dataItem: any) => dataItem[value]),
          enumNames: data.map((dataItem: any) => dataItem[label]),
        });
      });
    } else if (type === 'data') {
      if (isTable) {
        // table一般是直接绑定一个接口，该接口所以数据都是给table用
        field.forEach((filedItem) => {
          values = { ...values, [filedItem]: data };
        });
      } else {
        // 只把需要赋值的字段拼接过去，设计时已处理bind字段关联
        values = { ...values, ...pick(data, field) };
      }
    }
  });
  // 详情接口业务产生的数据最后拼接
  values = { ...values, ...detail };
  // 里面有类型校验，后续注意看怎么处理
  // debugger;
  // 我们有watch监听字段，当watch字段在接口提前触发更新的时候，formData已经更新了一轮，注意不要覆盖，数据的优先级后续考虑处理下，默认如果是同一个字段远程的覆盖联动的结果
  values = { ...form.getValues(), ...values };
  form.setValues(values);
  setFalse();
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, [id]);
```

整体思路

1：根据前面二步已经整理出了远程的数据接口配置

2：拼接 bindData 即前面二步整理的 系统变量等

3：对所有的 remoteDatas 发起远程接口请求

- 3.1：如果是 form 直接把返回的数据拼接过去，设计时数据绑定的时候同步了 bind 字段

- 3.2： 如果是 select,获取远程数之后通过 form.setSchemaByPath 更新对应的 enum 配置等

- 3.3 如果是 data 数据绑定： table 组件的情况下直接把返回值赋值给 table 组件，其他非 table 组件，从接口返回数据 pick 抽取绑定了该接口的字段进行数据绑定

4：formBusinessData.businessData 业务详情数据赋值拼接

5：拿到当前实时的值，最初拼接上面整理的所有值 通过 form.setValues(values); 进行赋值处理

> formBusinessData.businessData 产生的表单详情业务数据、和表单配置的远程数据的赋值优先级，以及联动的一些特殊情况的赋值，需要根据实际情况再验证下，不然可能存在覆盖的情况

## 数据联动

[设计时联动配置](/components/design/data-link)

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/utils/form/link.ts)

通过 [watch](https://x-render.gitee.io/form-render/advanced/watch) 一一设置对应的字段监听

```js
import { intersection, toNumber, toString, uniq } from 'lodash-es';
import type {
  DataLinkItem,
  DataLinkAction,
  DataLinkCondition,
  IOptions,
  TOptionsType,
} from '@/types';
import type { FormInstance } from 'form-render';

import { getValueByFormula } from './formula';
import { checkSelectNeedOptions } from './fields';

/**
 * @param type 类型
 * @description 把参数包装成需要转换的类型
 */
function formatValByType(val: any, type: 'string' | 'number' | 'boolean') {
  if (type === 'string') return toString(val);
  if (type === 'number') return toNumber(val);
  return !!val;
}

function checkConditionsIsTrue(conditions: DataLinkCondition[], formData: any) {
  // 为空直接满足条件
  if (!conditions || !conditions.length) return true;
  let conditionStr = '';
  conditions.forEach((item, index) => {
    // type类型暂不考虑select等特殊情况，统一直接根据类型转
    const { field, value, sign, type, combine } = item;
    // 多选数组的包含情况
    let conditionVal = false;
    if (sign === 'includes') {
      const intersectionData = intersection(formData[field] || [], value || []);
      if (intersectionData.length === (value || []).length) {
        conditionVal = true;
      }
    } else {
      // eslint-disable-next-line no-eval
      conditionVal = eval(`${formData[field]} ${sign} ${formatValByType(value, type)}`);
    }
    if (index === 0) {
      conditionStr += `${conditionVal}`;
    } else {
      conditionStr += `${combine === 'and' ? '&&' : '||'} ${conditionVal}`;
    }
  });
  // eslint-disable-next-line no-eval
  return eval(conditionStr);
}

/**
 * @description 根据联动条件获得联动的组装的结果
 */
export function getLinkResult(formData: any, schema: any): Record<string, Partial<DataLinkAction>> {
  // console.log('formData', formData);
  // console.log('schema', schema);
  const { dataLink }: { dataLink: DataLinkItem[] } = schema;
  // 只需要处理已经启用的联动
  const enableDataLink = dataLink.filter((item) => item.isEnable);
  // 联动条件可能存在互斥，简单覆盖,最后一条条件符合覆盖掉前面的
  const result: Record<string, Partial<DataLinkAction>> = {};
  enableDataLink.forEach((item: DataLinkItem) => {
    const { conditions = [], actions = [] } = item;
    // 判断条件是否满足
    const conditionsIsTrue = checkConditionsIsTrue(conditions, formData);

    actions.forEach((action) => {
      const { isEnable, isRequired, isVisible, field, formula, options, isClear } = action;
      // 条件满足，计算该条件联动的字段处理
      if (conditionsIsTrue) {
        result[field] = {
          isEnable,
          isRequired,
          isVisible,
          options,
        };
        const newValue = formula?.value ? getValueByFormula(formula.value, formData) : false;
        // 计算后的结果没有出错，需要进行值覆盖
        if (newValue !== false) {
          result[field].value = newValue as unknown as string;
          result[field].conditionsFieldIds = conditions.map((condition) => condition.field);
        }
      } else if (isClear) {
        // 条件不满足并且不满足需要清空字段值
        result[field] = {
          ...(result[field] ? result[field] : {}),
          // 清空这个值类型有待确定，有可能有空数组，后续还是需要严格区分字段类型
          value: '',
          conditionsFieldIds: conditions.map((condition) => condition.field),
        };
      }
    });
  });
  return result;
}

/**
 * @param schema 配置信息
 * @returns 所有联动条件涉及到的字段数组
 */
export function getConditionsFields(schema: any) {
  const { dataLink = [] }: { dataLink: DataLinkItem[] } = schema;
  // 只需要处理已经启用的联动
  const enableDataLink = dataLink.filter((item) => item.isEnable);
  const result: (string | number)[] = [];
  enableDataLink.forEach((item) => {
    const { conditions = [] } = item;
    conditions.forEach((condition) => {
      result.push(condition.field);
    });
  });
  return uniq(result);
}

/**
 * @description 转入options结构，然后转化为schema需要的 enum enumNames结构
 * @param {IOptions[]} options
 * @return {defaultVal：  any[], enumLabel: (string | number)[], enumValue: (string | number)[]  }
 */
export function optionsToEnum(options: IOptions[], mode: TOptionsType) {
  const defaultVal: any[] = [];
  const enumLabel: (string | number)[] = [];
  const enumValue: (string | number)[] = [];
  options.forEach((item: IOptions) => {
    if (item.default) {
      defaultVal.push(item.value);
    }
    enumLabel.push(item.label);
    enumValue.push(item.value);
  });
  const singleVal = defaultVal.length ? defaultVal[0] : undefined;
  return {
    defaultVal: mode === 'multiple' ? defaultVal : singleVal,
    enumLabel,
    enumValue,
  };
}

/**
 * @param {*} data 处理之后的需要连动杆的结果
 * @param {*} form form表单实列
 */
export function updateDataLinktoSchema(
  data: Record<string, Partial<DataLinkAction>>,
  form: FormInstance,
) {
  // console.log('整理后没有问题需要的联动数据:', data);
  // 使用form.schema去计算options等，如果直接使用  form.flatten（实时数据） 会导致部分options配置丢失
  const flattenData: IFlattenItem = flattenFilterPrefix(flattenSchema(form.schema) || {});
  const newFlattenData = cloneDeep(flattenData || {});

  Object.keys(data).forEach((field) => {
    const { isEnable, isRequired, isVisible, value, options = [] } = data[field] as DataLinkAction;
    const currentFieldValue = form.formData[field];
    // 如果传递form.flatten去计算，form.flatten是实时的值，如果存在刚开始单选按钮 = 1，多选选项1，2,3,4 全部显示，单选按钮显示 2，多选选项只显示 1， 2， 3 的联动，当切换的时候，单选1切到2,2再切会1会导致 1的联动的多选选项丢失（设置newSchema导致了最初的完整的多选选项丢失了,应该使用最原始完整的schema去过滤计算）
    const {
      result,
      options: optionsData = [],
      mode,
    } = checkSelectNeedOptions(field, [], flattenData);
    const newSchema: any = {
      disabled: !isEnable,
      required: isRequired,
      hidden: !isVisible,
    };
    if (result) {
      const { defaultVal, enumLabel, enumValue } = optionsToEnum(
        optionsData.filter((item) => options.includes(item.value)),
        mode as TOptionsType,
      );
      newSchema.enumNames = enumLabel;
      newSchema.enum = enumValue;
      // select选项没有值给个默认值
      if (typeof currentFieldValue === 'undefined') {
        newSchema.default = defaultVal;
      }
    }

    // console.log('更新后的数据', field, newSchema, value, form.schema);
    const currentFiledSchema = get(newFlattenData, field, {});
    set(newFlattenData, field, { ...currentFiledSchema, ...newSchema });
    // 直接设置setSchemaByPath，如果该条件下存在多个组件需要更新，那么可能导致schema直接覆盖，改用 setSchema全局改
    // form.setSchemaByPath(field, newSchema);

    // select目前支持联动配置options,但是不支持设置值
    if (!result) {
      form.setValueByPath(field, value);
    }
  });
  form.setSchema(newFlattenData);
}

export function getWatch(schema: any, form: FormInstance) {
  const conditionFieldIds = getConditionsFields(schema);
  const result: Record<
    string | number,
    (value?: any) => void | { handler: (value?: any) => void; immediate: boolean }
  > = {};

  conditionFieldIds.forEach((field: string | number) => {
    result[field] = () => {
      const data = getLinkResult(form.formData, schema);
      updateDataLinktoSchema(data, form);
    };
  });
  return result;
}

```

整体思路

1: getConditionsFields 获取已经启用的联动条件所有关联的字段列表

2: getLinkResult 根据联动条件，满足条件的进行收集，获得最终需要进行联动处理的数据集合， （后面的联动条件满足涉及到对同一个字段的处理，后一个会覆盖前一个）

3: updateDataLinktoSchema 根据第二步整理需要设置的联动数据进行遍历，setSchema 更新对应组件字段的 newSchema （setSchemaByPath 单个设置存在覆盖的情况）， setValueByPath 更新对应字段的值

> 联动功能值的设置支持 FX 表达式，通过 [expr-eval](https://www.npmjs.com/package/expr-eval) 进行处理，如果处理结果抛出异常，那么自动会把这个值的设置给忽略掉
