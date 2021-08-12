---
title: Upload 上传
order: 3
group:
  title: 运行时
  path: /run
---

大部分逻辑业务和之前的上传组件相同

[源代码](https://github.com/killdada/you-can-form-render/blob/master/src/pages/Form/Widgets/Run/Upload/index.tsx)

没有比较复杂的业务处理，大部分代码和之前的保持一致，简单介绍下部分功能即可

## 可配置项

![](http://qiniu.yenmysoft.com.cn/form-render-assets/components/upload.png)

## 公用、非公用

配置项，公用，公用和非公用上传的接口地址是不同的，部分代码处理稍微有一点不一样

```js
// 接口返回的数据有可能是内网、外网共存的数据，内网的话需要查询接口获取地址，外网不需要
const getFileList = async (fileList: UploadFileExtend[]) => {
  if (!fileList || fileList.length === 0) {
    return [];
  }
  const res: UploadFileExtend[] = [];
  fileList.forEach(async (file) => {
    const { fileName, url } = file;
    let realUrl = url;
    let realFileName = fileName;
    if (url?.includes('aliyuncs.com')) {
      // 外网地址 文件名去后面的后缀当文件名
      realFileName = path.basename(fileName);
    } else {
      // 内网地址通过接口获取真实url地址
      const { data } = await UploadService.getFileUrl(fileName as string);
      realUrl = data;
    }
    res.push({
      ...file,
      url: realUrl,
      fileName: realFileName,
      // 接口存储的是上一次操作的时候，这里重新刷新了内网图片的地址，有效期也需要刷新
      defaultTime: Date.now(),
    });
  });
  return res;
};
```

<Alert>

页面获取到历史上传的文件的时候，有可能配置的是公用上传了一次，然后又配置了非公用上传了一次，因此是存在公用、非公用混合的情况，虽然一般不会这么操作，因此在页面赋值的时候

简答判断了返回的 url 地址是不是包含 aliyuncs.com，包含说明这个文件是通过公用上传的，不包含就是文件存储到了内网地址，然后文件存储在内网的情况下，需要根据他的 fileName 获取真实的图片地址

</Alert>

## 检验错误

自定义组件单独提供检验方法，在提交的保存的时候触发各个组件自己的检验，目前看到 form-render 好像没有支持，issue 里面有看到类似的，虽然他提供了一个 addons.setErrorFields 去更改设置检验出错的地方，这个更改现在我们就必须是实时的去抛出错误（体验差了点）。而且使用的过程中发现存在 bug，因此参考部分代码，封装了一个类似的 setErrorFields 处理，详细实现 [参考](/code/model/fields)，后续考虑扩展实现自定义检验处理，只在表单点击提交的时候触发下检验即可

```js
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
  } else {
    removeErrorField(addons.dataPath);
  }
};
```
