---
title: Form 表单设计关联接口
order: 3
group:
  title: umi model
  path: /model
---

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/models/formModel.ts)

主要是绑定远程接口的时候需要的一些方法和属性，大部分接口直接跟以前的接口适配，接口地址相同返回结构相同，请求参数基本一样 （后续需要补充一部分请求参数）

## 属性

| 属性                 | 说明                       |
| -------------------- | -------------------------- |
| formCategorys        | 表单分类列表               |
| formList             | 所有表单列表               |
| databaseList         | 本地数据库列表             |
| databaseParamList    | 本地数据库对应接口字段列表 |
| thirdDatabaseList    | 第三方系统列表             |
| thirdDatabaseApiList | 第三方接系统接口列表       |

## 方法

| 方法                       | 说明                                                               | 参数                               |
| -------------------------- | ------------------------------------------------------------------ | ---------------------------------- |
| initFormDesignData         | 初始化设计时表单配置需要的一些远程、表单分类、列表等接口信息       | 当前绑定的远程接口信息             |
| updateThirdDatabaseApiList | 第三方系统选择改变，根据第三方系统应用 id 重新获取该应用的接口列表 | 第三方系统 appId                   |
| updateDatabaseParamList    | 本地数据库表选择改变，根据表名更新接口字段列表                     | 当前选中的本地数据库表名 tableName |

## 示例

### 1： initFormDesignData 初始化整体设计时配置需要的数据

```js
// src/pages/Form/Designer/index.tsx
const { initFormDesignData } = useModel('formModel', (model) => ({
  initFormDesignData: model.initFormDesignData,
}));

const { setCurrentSelectField } = useModel('fieldsModel', (model) => ({
  setCurrentSelectField: model.setCurrentSelectField,
}));

useEffect(() => {
  // 根据加载schema.data 表单数据绑定schema获取本地、第三方数据库数据
  initFormDesignData(schema?.formDataBind as RemoteBindData);
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);
```

### 2: updateThirdDatabaseApiList updateDatabaseParamList 根据选中的本地数据库表明，或者是第三方系统的 appId 更新对应需要的下拉数据

```js
// src/pages/Form/Widgets/Designer/components/RemoteBind/Content.tsx
const {
  databaseList,
  databaseParamList,
  thirdDatabaseList,
  thirdDatabaseApiList,
  updateThirdDatabaseApiList,
  updateDatabaseParamList,
} = useModel('formModel', (model) => ({
  databaseList: model.databaseList,
  databaseParamList: model.databaseParamList,
  thirdDatabaseList: model.thirdDatabaseList,
  thirdDatabaseApiList: model.thirdDatabaseApiList,
  updateThirdDatabaseApiList: model.updateThirdDatabaseApiList,
  updateDatabaseParamList: model.updateDatabaseParamList,
}));

const changeDataBase = async (tableName: string) => {
  // 表单数据绑定不需要马上绑定字段
  if (type === 'form') return;
  await updateDatabaseParamList(tableName);
};

const changeThirdDataBase = async (val: string | number) => {
  await updateThirdDatabaseApiList(val);
  formRef?.current?.setFieldsValue({ appInterId: '', appInterAddr: '' });
};

<Form.Item
  label="数据来源表"
  name="databaseTableName"
  rules={[{ required: true, message: '请选择数据来源表!' }]}
>
  <Select onChange={changeDataBase} showSearch optionFilterProp="children">
    {databaseList.map((item: IDataBase) => (
      <Option value={item.tableName} key={item.tableName}>
        {item.tableComment}
      </Option>
    ))}
  </Select>
</Form.Item>

<Form.Item
  label="第三方系统"
  name="appId"
  rules={[{ required: true, message: '请选择第三方系统!' }]}
>
  <Select onChange={changeThirdDataBase} showSearch optionFilterProp="children">
    {thirdDatabaseList.map((item: IThirdDataBase) => (
      <Option value={item.id} key={item.id}>
        {item.appName}
      </Option>
    ))}
  </Select>
</Form.Item>;
```
