---
title: DataLink 数据联动
order: 4
group:
  title: 设计时
  path: /design
---

数据联动配置

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/pages/Form/Widgets/Designer/DataLink/index.tsx)

[操作流程](/guide/business/data-link)

[ts 类型](/schema#datalinkitem-数据联动配置信息)

基于 [@ant-design/pro-form](https://procomponents.ant.design/components/modal-form) [@ant-design/pro-table](https://procomponents.ant.design/components/table?current=1&pageSize=5)

`部分注意事项`

1： 打开数据联动 新建、编辑弹窗的时候，刷新一次 umi model field 信息：联动条件、联动配置、联动 fx 表达式都需要使用表单的组件列表字段 select 数据

```js
// DataLink/EditModal/index.jsx
const { getFields } = useModel('fieldsModel', (model) => ({
  getFields: model.getFields,
}));

useImperativeHandle(ref, () => ({
  open: (data, type) => {
    // 打开弹窗的时候时候更新下组装的fields信息，后续子组件(condition、actions)不需要更新了，直接用
    getFields();
    setDataLink(data);
    setIsEdit(!!type);
    setEditShow(true);
    formRef?.current?.setFieldsValue(data);
  },
}));
```

2：联动条件的值，如果选择的条件是 select 选项，那么支持直接选择该 select 选项对应的值 （固定 options 支持，远程的话因为设计时运行时会话是不一样的。如果不通导致远程 options 在设计时出现问题，提示：之前的设计时好像是支持的，设计时也能拿到对应的远程 select 数据，这个待后续接入验证）

```js
// DataLink/EditModal/Conditions.jsx
{
  title: '值',
  dataIndex: 'value',
  tooltip:
    '如果是类select类型并且绑定数据来源为远程，设计时可能无法获取值对应的select,因为对应的请求可能依赖运行时环境，这里暂不支持远程数据来源下拉选择值！',
  renderFormItem: (_, { record }) => {
    const { result, options = [], mode } = checkFieldIsSelect(record?.field);
    if (result && options.length) {
      return (
        <Select placeholder="请选择值" mode={mode === 'multiple' ? 'multiple' : undefined}>
          {options.map((item) => (
            <Option value={item.value} key={item.value}>
              {item.label}
            </Option>
          ))}
        </Select>
      );
    }
    return <Input />;
  },
},
```

3：配置联动值，支持 fx 计算, fx 计算校验表达式合法性直接借助的是 [expr-eval](https://www.npmjs.com/package/expr-eval) , （之前的可能一些求和的一些功能待接入验证）

```js
// DataLink/EditModal/Actions.jsx
{
  title: '值',
  dataIndex: 'formula',
  renderFormItem: (_, { record }) => {
    if (record) {
      const { formula = { label: '', value: '' }, field, id } = record;
      const { result } = checkFieldIsSelect(field);
      if (result) return <></>;
      return (
        <div className="flex-between-center">
          <Input disabled value={formula.label} />
          <Button
            size="small"
            className="color-blue"
            type="text"
            onClick={() => {
              formulaRef.current?.open({ ...formula, id } as IFormula);
            }}
          >
            FX
          </Button>
        </div>
      );
    }
    return <></>;
  },
},

// DataLink/EditModal/Formula.jsx
import type { FC } from 'react';
import React, { useState, forwardRef, useImperativeHandle } from 'react';
import { useModel } from 'umi';
import { useSetState } from 'ahooks';

import { ModalForm, ProFormTextArea } from '@ant-design/pro-form';
import ProCard from '@ant-design/pro-card';

import type { FieldsData, IFormula, ModalRef } from '@/types';

import { checkFormula } from '@/utils';
import { message } from 'antd';

interface FormulaProps {
  onFinish: (value: IFormula) => void;
  ref: any;
}

const keyboard: (string | number)[] = [
  1,
  2,
  3,
  4,
  5,
  6,
  7,
  8,
  9,
  0,
  '+',
  '-',
  '*',
  '/',
  '(',
  ')',
  '.',
  '清除',
];

const Formula: FC<FormulaProps> = forwardRef<ModalRef<IFormula>, FormulaProps>((props, ref) => {
  const [show, setShow] = useState(false);

  const [formula, setFormula] = useSetState<IFormula>({ label: '', value: '' });

  const { fields } = useModel('fieldsModel', (model) => ({
    fields: model.fields,
  }));

  useImperativeHandle(ref, () => ({
    open: (data) => {
      setFormula(data);
      setShow(true);
    },
  }));

  // 点击公式
  const handleClickField = (item: FieldsData) => {
    setFormula({
      label: formula.label + item.label,
      value: formula.value + item.value,
    });
  };

  // 点击小键盘
  const handleClickSign = (item: string | number) => {
    if (item === '清除') {
      setFormula({
        label: '',
        value: '',
      });
    } else {
      setFormula({
        label: formula.label + item,
        value: formula.value + item,
      });
    }
  };

  const onFinish = async () => {
    // 空字符串不需要检验，直接合法
    if (!formula.value.length || checkFormula(formula.value)) {
      props.onFinish(formula);
      return true;
    }
    message.warn('表达式不合法！');
    return false;
  };

  return (
    <ModalForm
      width="800px"
      title="公式设置"
      visible={show}
      onVisibleChange={setShow}
      validateTrigger="onSubmit"
      layout="horizontal"
      modalProps={{
        forceRender: true,
        destroyOnClose: true,
        wrapClassName: 'data-link-modal-formula',
      }}
      onFinish={onFinish}
    >
      <ProCard split="vertical">
        <ProCard
          title="可选字段"
          className="data-link-modal-formula-fields"
          headerBordered
          colSpan="30%"
        >
          <ul>
            {fields.map((item) => {
              return (
                <li onClick={() => handleClickField(item)} key={item.value}>
                  {item.label}
                </li>
              );
            })}
          </ul>
        </ProCard>
        <ProCard title="小键盘" headerBordered className="data-link-modal-formula-keyboard">
          <ul>
            {keyboard.map((item) => {
              return (
                <li key={item} onClick={() => handleClickSign(item)}>
                  {item}
                </li>
              );
            })}
          </ul>
          <ProFormTextArea
            labelAlign="right"
            name="text"
            label="表达式"
            disabled
            fieldProps={{
              value: formula.label || '',
            }}
          />
        </ProCard>
      </ProCard>
    </ModalForm>
  );
});

export default Formula;


// src/utils/form/formula.ts
import { Parser } from 'expr-eval';

/**
 * @description 检查传入的公式是否是合法的公式
 * @param {string} param 传入的表达式字符串
 * @return {*}  {boolean}
 */
export function checkFormula(param: string): boolean {
  const parser = new Parser();
  try {
    parser.parse(param);
  } catch (error) {
    return false;
  }
  return true;
}

```
