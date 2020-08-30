---
layout:     post
title:      "Umi Prompt"
subtitle:   ""
date:       2020-08-30 17:58:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - react
---

在后台管理系统中，大多数页面都是数据录入界面，因此非常有必要当用户离开页面时弹出提示，让用户确认操作，避免误操作丢失未保存数据。自己实现会有很多问题，可以使用 Umi 封装的 Prompt 来实现。

Prompt 有两个属性

* message

字符串类型的提示，或者一个返回字符串或布尔值的函数，如果是一个函数，只要返回 true，页面就会继续跳转，返回字符串则会弹出提示。

* when

布尔值，可以是页面的一个变量，当变量为 false 时则页面离开时会弹出提示。 true 时页面离开则不会弹出提示。

#### 使用示例

* 直接使用，每次离开都提示

```javascript
import { Prompt } from 'umi';

export default () => {
  return (
    <div>
      {/* 用户离开页面时提示一个选择 */}
      <Prompt message="你确定要离开么？" />
    </div>
  );
};
```

* 根据条件选择弹出提示

```javascript
import { Prompt } from 'umi';

export default () => {
  return (
    <div>
      {/* 根据条件选择弹出提示 */}
      <Prompt
      message={location => {
        console.log(location.pathname);
        return location.pathname === "/list" ? `您确定要跳转到首页么？` : true;
      }}
    />
    </div>
  );
};
```

* 监控变量选择是否提示

```javascript
import { Prompt } from 'umi';

export default () => {
  return (
    <div>
      {/* 根据一个状态来确定用户离开页面时是否给一个提示选择 */}
      <Prompt when={dataIsSaved}} message="您确定半途而废么？" />
    />
    </div>
  );
};
```

如果觉得浏览器默认的弹出框样式不好看，可以结合 when 属性使用 antd 的 Modal 组件定制。

```javascript
import React, { useState } from 'react';
import { PageContainer } from '@ant-design/pro-layout';
import { Modal } from 'antd';
import { Prompt, history } from 'umi';

export default (): React.ReactNode => {
  const [when, setWhen] = useState(true);

  return (
    <PageContainer>
      <Prompt
        when={when}
        message={location => {
          // 弹出 Modal 框
          Modal.confirm({
            content: 'are you sure ?',
            onOk: () => {
              // 用户确认时修改 when 状态
              setWhen(false);
              // 再次跳转页面（此处有个小问题，when还未修改成false，会二次弹框，可解决）
              history.push(location.pathname);
            },
          });
          return false;
        }}
      />
    </PageContainer>
  )
};
```
