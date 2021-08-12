---
title: Config 基础配置
order: 1
group:
  title: umi model
  path: /model
---

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/models/configModel.ts)

主要是放置一些全局的基础配置，目前设置

## isDesign

描述：标识是运行时、还是设计时。

设置：在各自设计时运行时入口里面注入

用途：区分设计时，运行时做一些处理，比如

```js
// src/Page/Form/Widgets/Run/Upload/index.tsx
const { isDesign } = useModel('configModel', (model) => ({
  isDesign: model.isDesign,
}));

const isDisabled = isDesign || disabled || readOnly || fileList.length >= 5;

return (
  <div className="fr-upload-mod">
    <Upload
      {...uploadProps}
      {...comProps}
      className="fr-upload-file"
      disabled={isDisabled}
    >
      <Button icon={<UploadOutlined />} disabled={isDisabled}>
        上传
      </Button>
    </Upload>
  </div>
);
```

设计时上传组件应该是 disabled
