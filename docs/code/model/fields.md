---
title: Fields 字段处理
order: 4
group:
  title: umi model
  path: /model
---

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/models/fieldsModel.ts)

根据 schema 组装处理所有的字段列表信息等，方便数据联动的时候选择字段（方便判断当前选中组件是不是 select 组件）等、处理自定义组件抛出的的 errorFields、保存当前选中节点的信息

## 属性

| 属性               | 说明                                                                    |
| ------------------ | ----------------------------------------------------------------------- |
| currentSelectField | 当前选中的组件的信息                                                    |
| fields             | 表单所有的字段列表                                                      |
| errorFieldsRef     | errorFields useRef 处理 errorFilelds 的优化，因为可能会频繁的更新这个值 |

## 方法

| 方法                  | 说明                                                                                  | 参数                        |
| --------------------- | ------------------------------------------------------------------------------------- | --------------------------- |
| setCurrentSelectField | 保存更新当前选中组件信息                                                              | 当前选中组件信息            |
| getFields             | 根据 schema 获取方便处理的所有的字段列表结构                                          | -                           |
| setErrorFields        | 设置自定义组件的错误信息等                                                            | {name: dataPath, error: []} |
| removeErrorField      | 移除自定义组件的检验错误                                                              | dataPath                    |
| checkFieldIsSelect    | 通过传入的 id 检查这个字段是否是类 select 字段,如果是类 select 字段返回对应的 options | 选择的组件 id               |

## 示例

### 1: 监听 onCanvasSelect 方法设置当前选中组件信息

```js
// src/pages/Form/Designer/index.tsx
// 更新当前节点选中的信息
const onCanvasSelect = (data: any) => {
  setCurrentSelectField(data);
};
<Generator
  extraButtons={extraButtons}
  defaultValue={schema}
  settings={defaultSettings}
  commonSettings={defaultCommonSettings}
  globalSettings={globalSetting}
  widgets={widgets}
  ref={generatorRef}
  onCanvasSelect={onCanvasSelect}
/>;
```

### 2: getFields 获取表单组件字段列表，方便用户设置联动条件、联动功能、fx 选择对应字段,或者其他的一些便利性功能

```js
// 1 设置

// src/pages/Form/Widgets/Designer/DataLink/EditModal/index.tsx
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
// 2 使用
// src/pages/Form/Widgets/Designer/DataLink/EditModal/Actions.tsx
const { fields, checkFieldIsSelect } = useModel('fieldsModel', (model) => ({
  checkFieldIsSelect: model.checkFieldIsSelect,
  // 后续考虑可以过滤部分跟远程数据相关的类select选项标识他们不能选择该字段
  fields: model.fields,
}));

{
  title: '字段名称',
  dataIndex: 'field',
  width: '120px',
  valueType: 'select',
  fieldProps: {
    options: fields,
    allowClear: false,
  },
},
```

### 3: checkFieldIsSelect 检验选中的组件是否是 select 组件

```js
// src/pages/Form/Widgets/Designer/DataLink/EditModal/Actions.tsx
const { fields, checkFieldIsSelect } = useModel('fieldsModel', (model) => ({
  checkFieldIsSelect: model.checkFieldIsSelect,
  // 后续考虑可以过滤部分跟远程数据相关的类select选项标识他们不能选择该字段
  fields: model.fields,
}));

{
  title: 'options选项',
  dataIndex: 'options',
  width: '120px',
  tooltip:
    '如果是类select类型并且绑定数据来源为远程，设计时可能无法获取值对应的select,因为对应的请求可能依赖运行时环境，这里暂不支持远程数据来源下拉选择值！',
  renderFormItem: (_, { record }) => {
    const { result, options = [] } = checkFieldIsSelect(record?.field);
    if (result && options.length) {
      return (
        <Select mode="multiple" placeholder="请选择options选项">
          {options.map((item) => (
            <Option value={item.value} key={item.value}>
              {item.label}
            </Option>
          ))}
        </Select>
      );
    }
    return <></>;
  },
},
```

### 4: setErrorFields 设置自定义组件错误信息等， removeErrorField 移除错误

setErrorFields 是 fr-generator 里面 props.addons.setErrorFields 里面的一个小变种，之前在 upload 组件测试发现 官方提供的 setErrorFields
方法存在问题，即使设置了错误，后面 submit 的时候也会丢失 （好像是源码每次都是重新根据 schema 去检验，但是检验的时候没有处理用户设置的 setErrorFields）

```js
// 1 收集
// src/pages/Form/Widgets/Run/Upload/index.tsx
const updateErrorFields = (files: UploadFile[]) => {
  // 如果有一个status error | uploading 那么就直接抛出错误
  const hasError = files.some(
    (item) => item.status === 'error' || item.status === 'uploading',
  );
  if (hasError) {
    setErrorFields([
      {
        name: addons.dataPath,
        error: ['有文件上传类型错误或者正在上传中上传失败等！'],
      },
    ]);
    // addons.setErrorFields([
    //   { name: addons.dataPath, error: ['有文件上传类型错误或者正在上传中上传失败等！'] },
    // ]);
  } else {
    // addons.removeErrorField(addons.dataPath);
    removeErrorField(addons.dataPath);
  }
};

// 2 提交验证 src/pages/Form/Run/index.tsx
const beforeFinish = (params: ValidateParams) => {
  const errorFields = errorFieldsRef?.current;
  // 如果有收集到自定义的组件的错误，直接先抛出自定义的错误
  if (errorFields.length) {
    return errorFields;
  }
  return (params.errors as Error[]) || [];
};
```

> setErrorFields 目前是实时检验抛出的，在 table 里面也有这样操作，这样会有一定的缺陷，每次都是实时取检验合法性，有一定的性能问题，虽然使用了 ref 优化，后续考虑扩展自定义组件检验处理，只在用户提交按钮的时候再检验自定义组件
