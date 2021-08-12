---
title: DataBind 组件数据绑定
order: 1
group:
  title: 设计时
  path: /design
---

数据绑定

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/pages/Form/Widgets/Designer/Data/index.tsx)

[操作流程](/guide/business)

[ts 类型](/schema#idatabind-组件数据绑定)

该组件拆分成三个部分

![](http://qiniu.yenmysoft.com.cn/form-render-assets/components/data.png)

主要的核心是第二个部分 RemoteBindContent 公共部分，后面详细介绍，现在先提一下

## 其他注意事项

1: 非表格列的数据绑定再绑定的时候会同步更新当前组件 schema 的 bind 信息，这样运行时赋值可以直接进行赋值即可方便处理 （如果没有处理 bind 那么可能需要额外根据 $id 映射绑定下）

```js
const onFinish = async (values: IDataBind) => {
  if (props.onChange) {
    // 如果是远程接口记得同步下bind字段,这样远程表单接口请求下来，直接使用该表单接口返回的字段进行赋值即可，（id, bind都可以赋值所以可以这样处理，看文档 https://x-render.gitee.io/form-render/advanced/form-methods#%E4%BE%8B-3%EF%BC%9Abind 和 https://x-render.gitee.io/form-render/schema/schema#bind）
    if (!isTable && values.sourceMethod !== EDataBindType.system) {
      props.addons.onItemChange('bind', values.field);
    }
    if (
      isTable &&
      ((values.field || []) as TableDataBindSetting[]).some((item) => !item.fieldKey)
    ) {
      message.warn('请检查表格列数据值绑定key，值绑定key不能为空！');
      return false;
    }
    props.onChange(values);
    setShow(false);
    return true;
  }
  return true;
};

```

2：对于表格列的绑定，因为列配置、列数据绑定不在同一个组件，因此为了完备性检验，通常非 button table 操作列等如果没有绑定字段，filed （antd table dataIndex）的话一般这个时候这个列就是无意义的因为没有数据可以进行关联，提交的时候也不清楚这个列提交的是那个字段

```js
// src/utils/form/check.ts
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
```

3: 表格列字段绑定、列配置不在不同一个组件，当设计时进行配置关联，赋值的时候需要特别注意,虽然之前的配置里面还存在列 1 的绑定，但是用户在表格列配置的时候删除掉了，因此需要注意过滤掉，
始终以表格列配置的数据为准！！！ 表格列配置直接取当前选中的组件 currentSelectField 的 columnTableDataBind 字段

```js

const { currentSelectField } = useModel('fieldsModel', (model) => ({
  currentSelectField: model.currentSelectField,
}));

const selectId = currentSelectField.$id;
const isTable = currentSelectField.widget === CustomWidgetsTypes.mrTable;
// 删减了其他值，只留下需要的值
const tableColumnData: TableDataBindSetting[] = (
  currentSelectField.columnTableDataBind || []
).map((item) => ({
  id: item.id,
  rowTitle: item.rowTitle,
  rowType: item.rowType,
  fieldKey: '',
}));

// 需要处理下部分数据，数据在表格列已经删除的话，旧的fields是无意义的
const { field = [] } = props.value;
const res: TableDataBindSetting[] = [];
tableColumnData.forEach((item) => {
  const currentField = (field as Partial<TableSettingDataSourceType>[]).find(
    (fieldItem) => fieldItem.id === item.id,
  );
  const fieldKey = currentField ? currentField.fieldKey : '';
  res.push({
    id: item.id,
    rowTitle: item.rowTitle,
    rowType: item.rowType,
    fieldKey,
  });
});
fields = {
  ...props.value,
  field: res,
};
```

4：根据数据绑定配置情况，需要实时更新对应的比如本地数据库表的接口，和第三方系统的接口列表接口

```js
// 当前数据绑定接口配置的信息
const {
  sourceMethod,
  dataSourceMethod: dataSourceMethodLocal,
  databaseTableName: databaseTableNameLocal,
  appId,
} = props.value || {};
// 远程或者刚开始未初始化的时候，只要表单配置了绑定都需要处理更新本地数据库表字段
if (
  sourceMethod === EDataBindType.remote ||
  typeof sourceMethod === 'undefined'
) {
  // 需要根据实际情况获取下字段列表信息
  if (dataSourceMethod === ERemoteBindType.database && databaseTableName) {
    updateDatabaseParamList(databaseTableName);
  }
} else if (sourceMethod === EDataBindType.otherRemote) {
  // 其他远程情况，自行绑定接口字段，根据实际情况看要不要获取本地数据库字段列表，或者是第三方系统接口列表数据
  if (
    dataSourceMethodLocal === ERemoteBindType.database &&
    databaseTableNameLocal
  ) {
    updateDatabaseParamList(databaseTableName);
  }
  if (dataSourceMethod === ERemoteBindType.thirdDataBase && appId) {
    updateThirdDatabaseApiList(appId);
  }
}
```

## 核心组件，远程接口绑定

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/pages/Form/Widgets/Designer/components/RemoteBind/index.tsx)

分别抛出了 RemoteBindModal 、 RemoteBindContent

其中 RemoteBindContent 属于 RemoteBindModal modal 里面的主体内容，RemoteBindContent 的用途是因为 dataBind 数据绑定组件，之前该组件已经有了自己的弹窗 modal。后续为了扩展 dataBind 组件让他支持直接绑定其他远程接口 （一个表单数据来源于多个接口），因此就直接把旧的 RemoteBindModal 里面的 content 单独抽离开来形成 RemoteBindContent，该组件可以直接作为 modalForm 的主体内容嵌入到 dataBind 组件 （因为主体 content 内容是一致的）

ps: 后续可以考虑 data modal 去掉，统一都是在 RemoteBindModal 里面处理，（最初因为是扩展功能，简单的就把 content 抽离了下）

`注意事项`

1: select 选项绑定的时候，如果最初没有任何值，但是因为拖动 select 的时候 select 配置里面有默认的 enum，因此需要注意填充下默认值

```js
// type = select的时候，没有绑定来源方式的时候填充默认值
const getDefaultSelectData = (
  value: RemoteBindData,
  formData: any,
  type: BindComponentType,
) => {
  if (!value?.dataSourceMethod && type === 'select') {
    const { enum: enums = [], enumNames = [] } = formData || {};

    const options: IOptions[] = enums.map(
      (item: string | number, idx: number) => {
        // 直接取相同enumNames 下标作为label，取不到就用这个value当label
        const label = enumNames[idx] || item;
        // 新建生成id的是string,模板setting里面的是number,统一转string，不然后续判断重复的时候类型可能需要额外处理
        return { id: idx.toString(), label, value: item, default: false };
      },
    );

    return {
      dataSourceMethod: ERemoteBindType.fixed,
      options,
    };
  }
  return {};
};
```

2: 更改全局表单配置的时候,因为全局表单配置每一个组件都可以绑定这个接口里面的字段，如果用户更改了全局的表单配置接口的时候需要弹窗提醒下用户检查之前绑定这个接口的组件，让用户记得重新更改字段，毕竟接口改了，原先绑定的字段可能就对不上了。（后续可以做的更完善点，提示下现有那些组件已经绑定了旧的全局表单接口，现在只是简单判断接口是否变更了）

```js
// 修改数据绑定当牵扯到对应接口地址变更的情况下，需要弹窗提示，提醒用户之前绑定的字段手动重新绑定
const checkNeedConfirm = (val: RemoteBindData, originVal?: RemoteBindData) => {
  // 刚开始组件最初原始值为undefined的情况，不需提示
  if (typeof originVal === 'undefined') {
    return false;
  }
  if (val.dataSourceMethod !== originVal.dataSourceMethod) {
    // 数据库来源不同肯定接口不同
    return true;
  }

  if (val.dataSourceMethod === ERemoteBindType.database) {
    // 绑定的表名不同，需要重新绑定
    if (val.databaseTableName !== originVal.databaseTableName) {
      return true;
    }
  }

  if (val.dataSourceMethod === ERemoteBindType.thirdDataBase) {
    // 第三方系统不同，说明接口不同
    if (val.appId !== originVal.appId) {
      return true;
    }
    // 第三方系统接口绑定名称不同，地址不同需要重新绑定
    if (val.appInterId !== originVal.appInterId) {
      return true;
    }
  }
  return false;
};

const onFinish = async (values: RemoteBindData) => {
  if (type === 'select') {
    // console.log(values,'values')
    onFinshSelect(values);
    return;
  }
  // 对比下originVal，values的值，如果发现 路径接口变了（之前字段绑定了接口字段的一般需要重新绑定），提交的时候给个确认弹窗，提醒用户重新绑定各个字段的值
  const result = checkNeedConfirm(values, originVal);
  if (result) {
    Modal.confirm({
      content:
        '接口地址更改以后，请检查更新现有表单字段绑定！您确定需要更改配置吗？',
      onOk() {
        setShow(false);
        if (props.onChange) {
          props.onChange(values);
        }
      },
      onCancel() {
        //
      },
    });
  } else {
    setShow(false);
    if (props.onChange) {
      props.onChange(values);
    }
  }
};
```

3：如果是 select 组件的选项绑定，并且绑定的是固定 option 记得直接同步更新 `enumNames` `enum` `default`, 其他远程 options 远程可能拿不到值（如果能拿到那么也可以同步下·）

```js
// 同步更改对应的enum enumNames字段,(固定方式需要，其他方式因为跟实际接口相关，设计时无法拿到数据所有不需要更新)
const changeSchemaData = (values: RemoteBindData) => {
  const { dataSourceMethod, options = [] } = values;
  if (dataSourceMethod === ERemoteBindType.fixed) {
    const { defaultVal, enumLabel, enumValue } = optionsToEnum(options, comType as TOptionsType);
    props.addons.onItemChange('enumNames', enumLabel);
    props.addons.onItemChange('enum', enumValue);
    props.addons.onItemChange('default', defaultVal);
  } else {
    // 清空默认值， enum enumnames暂不需要清空，不然页面选项都是空白的
    props.addons.onItemChange('default', undefined);
  }
};
```
