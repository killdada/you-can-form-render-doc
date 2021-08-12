---
title: Table 表格
order: 2
group:
  title: 运行时
  path: /run
---

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/pages/Form/Widgets/Run/Table/index.tsx)

基于 [Pro-Components @ant-design/pro-table](https://procomponents.ant.design/components/table?current=1&pageSize=5) 封装扩展

## 生成展示列

根据表单配置 columnTableDataBind 绑定的列配置信息生成对应的列展示，如果表格禁用了删除列，那么最后就没操作那一列

```js
const generateColumnData = () => {
  let colList: TableSettingDataSourceType[] = columnTableDataBind;
  if (colList && colList.length) {
    // 过滤不显示
    colList = colList.filter((data: TableSettingDataSourceType) => data.isShow);
    const tempColumns: ProColumns<TableSettingDataSourceType>[] = colList.map(
      (item: TableSettingDataSourceType) => {
        return generatorColumnItem(item);
      },
    );

    if (hasDel) {
      tempColumns.push({
        title: '操作',
        valueType: 'option',
        render: () => {
          return null;
        },
      });
    }
    setColumns(tempColumns);
  }
};
```

```js
// 根据列配置id获取当前数据绑定的时候table列绑定的接口key值，即table dataIndex
const getDataIndex = (id: string | number) => {
  const fieldItem = fields.find((item) => item.id === id);
  if (fieldItem) {
    return fieldItem.fieldKey;
  }
  return false;
};

// 生成column的每个类型
const generatorColumnItem = (
  item: TableSettingDataSourceType = {} as TableSettingDataSourceType,
) => {
  const {
    id,
    isRequired,
    rowTitle,
    isDisabled,
    isMultiple,
    rowType,
    optionState = {} as SelectBindData,
  } = item;

  // 列配置，组件是否必填
  const rules = isRequired
    ? [
        {
          required: isRequired,
          message: tableComMessage[rowType as keyof typeof tableComMessage],
        },
      ]
    : [];

  // 根据数据绑定组件获取对应组件绑定的id key
  const dataIndex = getDataIndex(id);

  const columnItem: ProColumns = {
    title: rowTitle,
    formItemProps: () => {
      const formItemProps: any = {
        rules,
      };
      return formItemProps;
    },

    fieldProps: () => {
      // 列配置是否禁用
      const filedPorps: any = {
        disabled: isDisabled,
      };
      // 列配置select是否多选单选
      if (rowType === TableColumnComType.select) {
        filedPorps.options = optionState?.options || [];
        if (isMultiple) {
          filedPorps.mode = 'multiple';
        }
      }
      return filedPorps;
    },
  };

  // 部分没有绑定key的可能是无意义的因为提交的时候不知道提交那个字段key，按钮是正常的，因此最后在设计时处理最后强检验下，除了按钮类型其他table字段必须绑定对应key，这里暂时不管，只处理已经绑定的key
  if (dataIndex) {
    columnItem.dataIndex = dataIndex;
  }

  // 根据配置的组件类型rowType，自动映射使用 pro-table里面的 valueType 内置类型映射
  if (rowType === TableColumnComType.button) {
    columnItem.renderFormItem = () => {
      return <Button type="primary">{item.rowTitle}</Button>;
    };
  } else {
    columnItem.valueType = componentByRowType[
      item.rowType as keyof typeof componentByRowType
    ] as ProFieldValueType;
  }

  // 列，select选项配置远程接口数据
  if (
    rowType === TableColumnComType.select &&
    optionState.dataSourceMethod &&
    optionState.dataSourceMethod !== ERemoteBindType.fixed
  ) {
    columnItem.request = async () => {
      try {
        const {
          query,
          value: valueKey = '',
          label = '',
        } = getApiQuery({ data: optionState, id: $id, type: 'select' });

        const { data = [] } = await FormService.fetchRemoteFormData(query);
        return data.map((dataItem: any) => ({
          value: dataItem[valueKey],
          label: dataItem[label],
        }));
      } catch (error) {
        // eslint-disable-next-line no-console
        console.error(
          `请求table select数据产生错误请检查,${error.message || error.msg || error}`,
        );
        return [];
      }
    };
  }
  return columnItem;
};
```

<Alert type="info">

getDataIndex 方法，如果是 table 组件，数据绑定组件会要求给每一个列配置对应的数据 key (button 按钮除外)，因此需要把这个配置的 key 赋值给 dataIndex 也就是 table 组件对应绑定的字段

select 远程数据绑定，直接借用 pro-table 组件的 request 属性配置进行处理

整体的表格赋值来源 dataSource 来源于表单入口的设置 即 props.value

</Alert>

## 列的默认值处理

部分组件列配置的时候可能存在默认值，比如 select 组件，选择固定选项，并且有默认值设定的时候,目前 select 有这个需求，后续其他组件可能也有

```js
// 获取form的默认展示值
const getFormDefaultData = ({
  rowType,
  optionState = {} as SelectBindData,
  isMultiple,
}: TableSettingDataSourceType) => {
  let initVal;
  if (rowType === TableColumnComType.select) {
    if (optionState.dataSourceMethod === ERemoteBindType.fixed) {
      const { defaultVal } = optionsToEnum(
        optionState?.options || [],
        isMultiple ? 'multiple' : 'single',
      );
      initVal = defaultVal;
    } else if (isMultiple) {
      initVal = [];
    }
  }
  return initVal;
};

// 获取每一个列的初始化默认数据
const getDefaultData = useCallback(() => {
  const result = {} as any;
  fields.forEach((item) => {
    // item里面只有部分字段，我们需要完整的字段用来设置默认值，从columnTableDataBind里面拿
    const optionInfo = columnTableDataBind.find(
      (col: TableSettingDataSourceType) => col.id === item.id,
    );
    if (optionInfo) {
      const initVal = getFormDefaultData(optionInfo);
      if (item.fieldKey) {
        result[item.fieldKey] = initVal;
      }
    }
  });
  return result;
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, [columnTableDataBind, JSON.stringify(fields)]);

const defaultBindData = useMemo(() => getDefaultData(), [getDefaultData]);

// 新增的时候 defaultBindData处理
<EditableProTable
    columns={columns}
    rowKey="id"
    actionRef={actionRef}
    value={dataSource}
    controlled
    bordered={true}
    recordCreatorProps={
      hasAdd
        ? {
            newRecordType: 'dataSource',
            record: () => ({
              ...defaultBindData,
              id: Date.now(),
            }),
          }
        : false
    }
    editable={{
      type: 'multiple',
      form,
      editableKeys,
      actionRender: (row, config, defaultDoms) => {
        return [defaultDoms.delete];
      },
      onValuesChange: (record, recordList) => {
        setDataSource(recordList);
        props.onChange(recordList);
      },
      onChange: setEditableRowKeys,
    }}
  />
```
