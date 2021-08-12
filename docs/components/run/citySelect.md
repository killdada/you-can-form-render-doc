---
title: CitySelect 城市选择
order: 1
group:
  title: 运行时
  path: /run
---

[源码](https://github.com/killdada/you-can-form-render/blob/master/src/pages/Form/Widgets/Run/CitySelect/index.tsx)

基于 [antd cascader](https://ant.design/components/cascader-cn/) 进行封装, 整体交互上和之前不一样，然后因为 cascader 也没有同时支持远程搜索功能，因此原先的搜索上体验会差点，
如果需要保留原先的交互实现，用三个 select 去实现，那么需要额外再进行封装，这里暂时维持使用 cascader 处理

```js
import type { FC } from 'react';
import React, { useEffect } from 'react';
import { useSafeState } from 'ahooks';
import { useModel } from 'umi';

import { Cascader } from 'antd';

import { CityService } from '@/service';
import type { ICity } from '@/types';
import type { CascaderOptionType, CascaderValueType } from 'antd/es/cascader';

const formatCityRes = (data: ICity[] = [], isLeaf: boolean): CascaderOptionType[] => {
  const res: CascaderOptionType[] = [];
  data.forEach((item: ICity) => {
    res.push({
      label: item.name,
      value: item.id,
      isLeaf,
    });
  });
  return res;
};

interface formMatOptionsParams {
  provinceId: number;
  cityId: number;
  provinces: ICity[];
  citys: ICity[];
  areas: ICity[];
}

// 三级联动省市区组装
const formatOptions = ({
  provinceId,
  cityId,
  provinces,
  citys,
  areas,
}: formMatOptionsParams): CascaderOptionType[] => {
  const res: CascaderOptionType[] = [];
  provinces.forEach((item: ICity) => {
    const provinceItem: CascaderOptionType = { label: item.name, value: item.id, isLeaf: false };
    if (item.id === provinceId) {
      provinceItem.children = citys.map((city: ICity) => {
        const cityItem: CascaderOptionType = { label: city.name, value: city.id, isLeaf: false };
        if (city.id === cityId) {
          cityItem.children = formatCityRes(areas, true);
        }
        return cityItem;
      });
    }
    res.push(provinceItem);
  });
  return res;
};

const MrCitySelect: FC<Generator.CustomComponentsProp<number[]>> = (props) => {
  const { isDesign } = useModel('configModel', (model) => ({
    isDesign: model.isDesign,
  }));
  const { value = [], onChange, disabled, readOnly, hidden, schema = {} } = props;
  const { props: comProps } = schema;

  const [options, setOptions] = useSafeState<CascaderOptionType[]>([]);

  useEffect(() => {
    if (!isDesign) {
      if (value[2]) {
        // eslint-disable-next-line @typescript-eslint/no-use-before-define
        fetchAllData();
      } else {
        // eslint-disable-next-line @typescript-eslint/no-use-before-define
        getProvinceList();
      }
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [isDesign]);

  const getProvinceList = async () => {
    const { data } = await CityService.fetchProvinces();
    setOptions(formatCityRes(data, false));
  };

  const fetchAllData = async () => {
    const provinceId = value[0];
    const cityId = value[1];
    const [provinceData, cityData, areaData] = await Promise.all([
      CityService.fetchProvinces(),
      CityService.fetchCitysByProvinceId(provinceId),
      CityService.fetchAreasByCityId(cityId),
    ]);
    setOptions(
      formatOptions({
        provinceId,
        cityId,
        provinces: provinceData.data,
        citys: cityData.data,
        areas: areaData.data,
      }),
    );
  };

  const loadData = async (selectedOptions?: CascaderOptionType[]) => {
    if (!selectedOptions || !selectedOptions.length) return;
    const targetOption = selectedOptions[selectedOptions.length - 1];
    targetOption.loading = true;
    let data: ICity[] = [];
    if (selectedOptions.length === 2) {
      const { data: dataRes } = await CityService.fetchAreasByCityId(targetOption.value || '');
      data = dataRes;
    } else if (selectedOptions.length === 1) {
      const { data: dataRes } = await CityService.fetchCitysByProvinceId(targetOption.value || '');
      data = dataRes;
    }
    targetOption.loading = false;
    targetOption.children = formatCityRes(data, selectedOptions.length === 2);

    setOptions([...options]);
  };

  const onChangeCity = (selectVal: CascaderValueType) => {
    // 只有选中三级的时候才回填数据给组件
    if (selectVal.length === 3) {
      onChange(selectVal);
    }
  };

  if (hidden) return null;

  return (
    <Cascader
      style={{ width: '100%' }}
      disabled={isDesign || disabled || readOnly}
      options={options}
      loadData={loadData}
      value={value}
      onChange={onChangeCity}
      changeOnSelect
      {...comProps}
    />
  );
};

export default MrCitySelect;

```

<Alert>
整体实现跟 Cascader loadData 提供的demo差不多，不过多介绍

需要注意的是以前的省市区是三个 select 并列，并且每一个 select 都需要单独配置数据来源，目前使用 Cascader 直接内置了这个数据绑定来源，不需要额外配置
</Alert>
