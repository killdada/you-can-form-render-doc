---
title: Schema 表单配置
order: 2
group:
  title: umi model
  path: /model
---

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/models/schemaModel.ts)

初始化设置 schema 配置等，保存更新 schema 等

## 属性

| 属性             | 说明                                                                                                                    |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------- |
| generatorRef     | fr-generator 库注入的 useRef，大部分在内部使用，没有外部调用                                                            |
| schema           | 设计时，运行时需要的配置 schema 信息，每次编辑的时候并没有实时同步 schema，需要拿实时的值请通过下方的 getFormSchemaData |
| formBusinessData | 以前的完整的表单运行时获取的接口数据（表单 schema 和其他字段的组合体）                                                  |

## 方法

| 方法                     | 说明                                                                                   | 参数                                                  |
| ------------------------ | -------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| initFormSchema           | 初始化请求远程接口 schema 数据                                                         | 表单 id                                               |
| getFormSchemaData        | 从 schema 里面取数据，支持 `lodash.get` (实时数据)                                     | key                                                   |
| getFormBusinessDataByKey | 从 formBusinessData 取数据，支持 `lodash.get` ，一般用来运行时取 schema 之外的一些数据 | key, defaultVal                                       |
| saveSchemaData           | 保存更新 schema 表单配置数据                                                           | id: 表单 id, schemaData: 自定义需要更新的 schema 数据 |

## 示例

### 1：设计时通过 getFormSchemaData 获取实时的 schema 和当前的 schema 对比，判断是否需要提示保存

```js
// src/pages/Form/Designer/index.tsx
const { generatorRef, saveSchemaData, getFormSchemaData, schema } = useModel(
  'schemaModel',
  (model) => ({
    schema: model.schema,
    generatorRef: model.generatorRef,
    saveSchemaData: model.saveSchemaData,
    getFormSchemaData: model.getFormSchemaData,
  }),
);

/* 用户离开页面时如果本地配置没有保存，提示一个选择 */
<Prompt
  message={() => {
    return !isEqual(getFormSchemaData(), schema)
      ? '您的配置暂未保存，你确定要离开么？'
      : true;
  }}
/>;
```

> getFormSchemaData 方法获取的时候是通过 fr-generator 提供的 generatorRef.current.getValue()方法，打印的时候发现会额外返回一些 undefined 数据，导致这里的 isEqual 判断出现错误，因此 getFormSchemaData 过滤掉了第一层的 undefined 数据

### 2：运行时 通过 getFormBusinessDataByKey 获取对应的表单修改历史

```js
// src/pages/Form/Run/ModifyHistory/index.tsx 运行时表单修改历史
const { getFormBusinessDataByKey } = useModel('schemaModel', (model) => ({
  getFormBusinessDataByKey: model.getFormBusinessDataByKey,
}));

const editRecord = getFormBusinessDataByKey('processRecord.editRecord', false);
if (!editRecord) return null;
```

### 3：设计时 saveSchemaData 保存表单配置

```js
// src/pages/Form/Designer/index.tsx
const { saveSchemaData, schema } = useModel('schemaModel', (model) => ({
    saveSchemaData: model.saveSchemaData,
    schema: model.schema as PlaygroundState,
}));

const extraButtons = [
  true,
  true,
  true,
  true,
  // 新增官方的playground 方便用户使用
  {
    text: '高级编辑',
    type: 'primary',
    title: '推荐比较清楚配置对应的关系再使用该功能。',
    onClick: () => {
      history.push(`/playground/${id}`);
    },
  },
  {
    text: '保存',
    type: 'primary',
    onClick: () => saveSchemaData(id),
  },
];

// src/pages/Form/Designer/Playground/index.tsx
const onSave = async () => {
  const schemaData = schemaStr2Json(playgroundRef.current?.schemaStr as string, true);
  saveSchemaData(id, schemaData);
};

```

`特别注意事项`

保存的时候做了一部分的 schema 检验处理，[详情见](/code/business/check)
