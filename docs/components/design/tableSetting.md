---
title: TableSetting 表格数据绑定
order: 6
group:
  title: 设计时
  path: /design
---

表格列配置

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/pages/Form/Widgets/Designer/TableSetting/index.tsx)

[操作流程](/guide/business/table)

[表格 ts 类型](/schema#tablesettingdatasourcetype-table-组件数据绑定)

基于 [@ant-design/pro-form](https://procomponents.ant.design/components/modal-form) [@ant-design/pro-table](https://procomponents.ant.design/components/table?current=1&pageSize=5)

`部分代码注意事项`

1: 表格列配置和列数据绑定时不同的组件，列数据绑定在数据绑定组件，因此当删除一个列的时候记得同步更新数据绑定 dataBind.field 信息

```js
const onFinish = async () => {
  await form.validateFields();
  let field: TableDataBindSetting[] = get(
    props,
    'addons.formData.dataBind.field',
    [],
  ) as TableDataBindSetting[];
  dataSourceOrigin.forEach((item) => {
    if (!dataSource.find((data) => data.id === item.id)) {
      // 该字段已经被删除，如果 dataBind.field有该字段 需要同步更新删除 dataBind.field
      field = field.filter((fieldItem) => fieldItem.id !== item.id);
    }
  });
  props.addons.onItemChange('dataBind.field', field);
  props.onChange(dataSource);
  return true;
};
```

2：列选择 Select，需要绑定选项来源，使用的是数据绑定组件渲染，[详情](/components/design/data-bind)

```js
{
  title: '选项来源',
  key: 'optionState',
  dataIndex: 'optionState',
  tooltip: '当选择的组件类型为下拉框时，可以开启选择options选项。',
  renderFormItem: (_, { record }) => {
    const { rowType, isMultiple = false, optionState = {} } = record || {};
    const isSelect = rowType === TableColumnComType.select;
    if (isSelect) {
      // 注意剔除掉onChange，不能用外部的fr-generator 提供的onChange，而应该直接用内部的formItem包裹的onChange （内部已经透传了）
      const { onChange, ...other } = props;
      const comPorps = {
        ...other,
        schema: {
          ...other.schema,
          // 注意需要标识是单选还是多选，固定选项的填写的时候需要用到这个字段
          comType: isMultiple ? 'multiple' : 'single',
        },
        value: optionState as SelectBindData,
      };
      return <RemoteBindModal isTable={true} type="select" {...comPorps} />;
    }
    return <></>;
  },
},
```
