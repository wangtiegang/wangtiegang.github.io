---
layout:     post
title:      "基于antd实现懒加载Select"
subtitle:   ""
date:       2020-05-10 15:23:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - Javascript
---

#### 背景及设计

在前端中，```<Select>``` 组件是使用非常频繁的一个组件，antd 的 Select 非常美观好用，但是在数据量非常大的场景下，也必须进行一些封装才能有好的体验。有些业务数据可以高达几万，比如员工，供应商等，虽然 Select 支持虚拟滚动技术，可以轻松加载这么大的数据，但是一次性向后端请求这么多数据非常耗时，造成用户等待，体验非常不好。如何解决这个问题呢？

我们知道 Select 组件的数据源是完全受控的，关键在于控制返回的数据，要解决怎么在返回选项少的情况下同时满足用户的选择需求。表面上看，用户选择什么我们无法预知，只能返回全部选项任其选择，其实不然。**分析用户行为可以知道，当选项超过一定数量又无法一眼找到自己的选项时，是一定会使用搜索寻找的，通过搜索缩小范围，直到找到目标选项**，利用这一点，我们可以设计一个懒加载的 Select 组件。

* 首先我们可以返回前100个选项，不管是否包含用户所需选项
* 当用户搜索时，如果匹配项大于100，则只返回前100个
* 用户不断精确搜索，最终会返回目标选项

在组件整个生命周期内，不管总共有多少选项，后端一次返回的选项都不会超过100个，这样组件第一次加载速度非常快，节约了大量资源和时间。

上面的思路没有问题，但是在实现的过程中我们还需要注意几个问题。

* 封装高阶组件时需要遵循受控组件思想
* ```Select``` 经常搭配 ```<Form>``` 使用，封装后的组件也必须具有受控的 ```value``` 和 ```onChange``` 属性
* 除部分必须覆盖的属性，其他属性需跟原生 Select 一致，可以受外部控制，透传给原生 Select
* 选中项由外部传入时，必须判断当前选择项是否包含选中项，不包含则需要后端请求数据
* 在多选模式下，用户搜索时需将已选项保留并合并到搜索结果中，保证选项是包含已选项的

#### 实现

在上面的设计指导下，具体实现如下

```javascript
import React, { useState, useEffect } from 'react';
import { Select, Spin } from 'antd';
import debounce from 'lodash/debounce';
import difference from 'lodash/difference';

const { Option } = Select;

/**
 * 懒加载 Select ，适用于数据超大的下拉框
 * @param {Select 官方属性} props
 */
const LazySelect = props => {
  const { value, onChange, query } = props;
  // 清除 porps 中 query，避免控制台警告
  const selectProps = { ...props, query: undefined };

  const [selected, setSelected] = useState(value);
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);

  const getSelectedArray = obj => {
    let selectedValues = obj;
    // 如果是单选，将值封装为数组
    if (obj && obj instanceof Array === false) {
      selectedValues = [obj];
    }
    return selectedValues;
  };

  // 添加 300 毫秒防抖
  const handleQuery = debounce(async param => {
    setLoading(true);
    const resp = await query(param);
    setData(resp);
    setLoading(false);
  }, 300);

  // 组件初始化时加载一次数据
  useEffect(() => {
    handleQuery({
      filter: '',
      selectedValues: getSelectedArray(value),
    });
  }, []);

  // 外部注入的 value 变化后，如果 value 在 data 中不存在，则加载数据
  useEffect(() => {
    setSelected(value);
    const dataKeys = data.map(item => item.value);
    const diff = difference(getSelectedArray(value), dataKeys);
    if (diff && diff.length > 0) {
      handleQuery({
        filter: '',
        selectedValues: getSelectedArray(value),
      });
    }
  }, [value]);

  // 搜索服务端异步加载
  const handleSearch = filter => {
    handleQuery({
      filter,
      selectedValues: getSelectedArray(selected),
    });
  };

  const handleChange = (newValue, option) => {
    setSelected(newValue);
    if (onChange) {
      // 将值通过 onChange 传递到外部
      onChange(newValue, option);
    }
  };

  return (
    <Select
      {...selectProps}
      value={selected}
      loading={loading}
      onSearch={handleSearch}
      onChange={handleChange}
      filterOption={false}
      showSearch
      showArrow
      notFoundContent={loading ? <Spin size="small" /> : null}
    >
      {data.map(d => (
        <Option key={d.value} title={d.label}>
          {d.label}
        </Option>
      ))}
    </Select>
  );
};

export default LazySelect;
```

以上就是具体实现了，后端需要配合返回相应数据，这块很简单，就不记录了。使用 ```LazySelect``` 非常简单，只需要传入一个查询数据函数。

```javascript
// 单选
<LazySelect query={fetchEmployee} />

// 多选
<LazySelect mode="multiple" query={fetchEmployee}>
```