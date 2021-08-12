---
title: Check 数据检验
order: 1
group:
  title: 功能实现
  path: /business
---

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/utils/form/check.ts)

目前主要是在设计时配置表单信息的时候进行了检验

```js
import { cloneDeep, set, get } from 'lodash-es';

import { IS_PHYSICAL_DEL } from '@/constants';
import type {
  FieldsData,
  IFlattenItem,
  ISchemaCommonProps,
  ISchemaPartial,
  TableDataBindSetting,
} from '@/types';
import { TableColumnComType } from '@/types';
import { FormTypes } from '@/types';

import { getAllFields, getDelFields } from './fields';
import { flattenSchema, idToSchema } from './tool';
import { checkArrayIsRepeat } from '../common';

/**
 * @description 提交保存的时候检查schema数据 $ids 是否存在空，是否存在重复id
 * @param {*} schema 传入的scheme数据
 * @return {*}  {{ msg: string; result: boolean }}
 */
function checkIds(flatten: IFlattenItem): { msg: string; result: boolean } {
  // 把schema 结构数据扁平化 (# 根，#/代表为空)
  const ids = Object.keys(flatten);
  const isHasEmpty = ids.some((item) => item === '#/');
  if (isHasEmpty) {
    return {
      msg: '请检查ID,ID不能为空！',
      result: false,
    };
  }

  // 这里的重复不是很友好，毕竟children下里面的字段还是有可能重复的，后续等数据结构复杂，再处理
  if (checkArrayIsRepeat(ids)) {
    return {
      msg: '请检查ID,ID不能重复！',
      result: false,
    };
  }
  return {
    msg: '检验id成功！',
    result: true,
  };
}

/**
 * @description 物理删除和非物理删除的不同,需要处理更新下保存的schema的结构
 * @param {{
 *   schema: ISchemaPartial;
 *   oldSchema: ISchemaPartial;
 *   fieldsData: FieldsData[];
 *   flattenData: IFlattenItem;
 * }} {
 *   schema, 新schema
 *   oldSchema, 旧schema
 *   fieldsData, 新字段列表，只是为了不想重复计算
 *   flattenData,新schema扁平对象结构，只是为了不想重复计算
 * }
 * @return {*}  ISchemaPartial
 */
function getNewSchemaData({
  schema,
  oldSchema,
  fieldsData,
  flattenData,
}: {
  schema: ISchemaPartial;
  oldSchema: ISchemaPartial;
  fieldsData: FieldsData[];
  flattenData: IFlattenItem;
}) {
  // 物理删除直接返回当前的即可
  if (IS_PHYSICAL_DEL) return schema;
  // 非物流删除，不能删除掉字段，只是给删除的字段打上标识 isDeleted 类似hidden效果
  const oldFlattenData: IFlattenItem = flattenSchema(oldSchema) || {};
  const oldFieldsData = getAllFields({}, oldFlattenData);
  // 获得删除的字段
  const delFields = getDelFields(fieldsData, oldFieldsData);
  if (delFields && delFields.length) {
    // 暂不考虑复杂嵌套，删除的字段实际已经没有任何其他意义，只是因为可能和流程强关联导致物理删除带来一堆问题，只要简单的把删除的字段信息拼接到新schema的
    const flatten = cloneDeep(flattenData);
    const oldFlatten = cloneDeep(oldFlattenData);

    delFields.forEach((fieldId) => {
      const id = `#/${fieldId}`;
      const isDeleted = get(oldFlatten, `${id}.schema.isDeleted`, false);
      if (!isDeleted) {
        // 现在的删除类似hidden导致可以删除多次，isDeleted标识下避免设置多次 （后续考虑编辑器fork一份）
        set(oldFlatten, `${id}.schema.hidden`, true);
        set(oldFlatten, `${id}.schema.isDeleted`, true);
        // label也标识下已删除,
        set(oldFlatten, `${id}.schema.title`, `${get(oldFlatten, `${id}.schema.title`, '')}(删)`);
      }
      flatten[id] = oldFlatten[id];
      // 别忘了跟节点的children push删除掉的节点
      flatten['#'].children.push(id);
    });
    return idToSchema(flatten, '#', true);
  }
  return schema;
}

// 检查table列有没有绑定字段，现在列配置和数据绑定是不同的二个入口，当新增一个列的时候保存此时可能用户还没有给该列绑定字段，列没有绑定字段大部分情况下是无意义的
function checkDesignTable(flatten: IFlattenItem = {}) {
  const ids = Object.keys(flatten) || [];
  let result = true;
  // eslint-disable-next-line no-restricted-syntax
  for (const id of ids) {
    const { schema = {} } = flatten[id] || {};
    const { columnTableDataBind = [], dataBind = {} } = schema as ISchemaCommonProps;
    const { field = [] } = dataBind;
    const hasEmpty = columnTableDataBind.some((item) => {
      // button不需要检验
      if (item.rowType !== TableColumnComType.button) {
        const fieldData = (field as TableDataBindSetting[]).find(
          (fieldItem) => fieldItem.id === item.id,
        );
        if (!fieldData || !fieldData.fieldKey) {
          return true;
        }
        return false;
      }
      return false;
    });
    if (hasEmpty) {
      result = false;
      break;
    }
  }
  return {
    result,
    msg: result ? '' : '请检查表格列数据绑定，部分列数据没有绑定key，这可能是无意义的！',
  };
}

/**
 * @description 组装还原适配以前的接口结构，并进行数据检验
 * @param {ISchemaPartial} [schema={}]
 * @return {result: boolean; msg?: string; data?: any }
 */
export function checkDesignForm(
  schema: ISchemaPartial = {},
  oldSchema: ISchemaPartial = {},
): { msg: string; result: boolean; data?: any } {
  // 把schema 结构数据扁平化 (# 根，#/代表为空)
  const flattenData: IFlattenItem = flattenSchema(schema) || {};

  // 先检验id
  const checkIdRes = checkIds(flattenData) || {};
  const { result } = checkIdRes;

  // 检验失败直接返回
  if (!result) return checkIdRes;

  // 开始检验其他数据
  const { formName, formType, formCategory, relFormId, formDesc, isNeedApprove } = schema;
  if (!formName) {
    return {
      result: false,
      msg: '表单名称不能为空！',
    };
  }
  if (!formCategory) {
    return {
      result: false,
      msg: '表单所属分类不能为空！',
    };
  }
  if (formType === FormTypes.approve && !relFormId) {
    return {
      result: false,
      msg: '审批表关联的表单不能为空！',
    };
  }

  // 检验表格数据
  const checkTableRes = checkDesignTable(flattenData) || {};
  // 检验失败直接返回
  if (!checkTableRes.result) return checkTableRes;

  const fieldsData = getAllFields({}, flattenData);
  const newSchema = getNewSchemaData({ schema, oldSchema, fieldsData, flattenData });

  // console.log('最新的schema结构：', newSchema);

  // 组装所有的字段列表数据,以前的formInfo.formStr
  const formStr = JSON.stringify(
    fieldsData.map((item) => ({
      formTitle: item.label,
      formDefineName: item.value,
    })),
  );

  const data = {
    formName,
    formType,
    formCategory,
    relFormId,
    formDesc,
    isNeedApprove,
    formStr,
    fileText: JSON.stringify(newSchema),
  };
  return {
    result: true,
    msg: '检验成功！',
    data,
  };
}

```

整体代码流程

1：检验组件 id 是不是为空，是不是存在重复， checkIds （没有考虑嵌套对象）

2：检验表单的一些基础信息填写等

3：checkDesignTable 检验表格数据列配置和数据绑定等

4：组装接口需要的结构

<Alert type="error">
特别注意：getNewSchemaData 方法会根据配置的是物理删除还是逻辑删除。历史原因，现有的表单组件移除的时候没有做物理删除，因为之前的表单配置等，有很多地方表单字段跟其他业务耦合度比较深，比如和流程等关联性，如果做物理删除，可能会导致已经关联的流程表单字段出现问题，导致流程无法继续。

因此新版本也没有做物理删除，即删除字段并不会彻底移除该字段。目前的做法是对比新旧 schema 配置的字段列表，识别出哪些字段是被删除了的，如果该字段被删除那么把这个字段对应的旧配置信息直接添加到新 schema 的末尾，并且增加 label 标识已删除，设置 hidden: true。

后续如果要做物理删除，只能完全重构流程和表单的交互关联逻辑
</Alert>
