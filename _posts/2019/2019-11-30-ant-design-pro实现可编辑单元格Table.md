---
layout:     post
title:      "ant design pro实现可编辑单元格Table"
subtitle:   "基于ant design pro v4 版本"
date:       2019-11-30 17:59:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - React
    - antd
    - JavaScript
---

ant design 是一个非常优秀的 react 前端组件库，如果使用ant design pro作为前端应用的基础就更方便了，很多后端管理项目都需要展示大量数据并且修改，通常都是使用 Table 组件来实现，并且编辑通常是单元格编辑，就像 Excel 一样。antd 提供了一个功能丰富的 Table 组件，但是有一个问题，就是没法通过组件的属性直接控制单元格编辑，这块需要自己实现，好在官网有个简单的例子，通过对例子的理解改造可以自己实现一个。

官方的例子实现思路是通过覆盖默认的行和单元格组件，单元格组件中绑定数据，在单元格 ```Cell``` 渲染时判断是否可编辑，如果可编辑则渲染一个 ```Input``` ，然后绑定数据和监听 ```onPressEnter``` 和 ```onBlur``` 事件来判断数据是否修改，并将数据返回至 ```state``` ，其中数据双向绑定是通过 ```Form.create()``` 注入单元格 ```form``` 属性实现的，监听函数是通过 ```Table``` 的 ```columns``` 属性传递过去的，在单元格 ```Cell``` 渲染时实现绑定。

官方的例子很简单，存在几个可以改进的地方：

* 所有组件都写在一个文件中
  
  > 可以拆分成几个文件，最终 export 一个组件，方便所有地方复用

* 数据直接保存在 ```state``` 中
  
  > 可以将数据保存到 _mock 文件中，模拟服务端从数据库中读取

* 只有新增和修改方法，并且修改后直接就保存了

  > 可以增加删除方法，并且增删改都只修改 dva 中的本地数据，当点击保存时再同步修改到服务器

代码实现
---

> 所有代码都在 ant design pro v4 版本的基础上添加

* 在项目的通用组件文件加下新建一个目录 ```EditableTable```，并新建如下文件
  
```EditableContext.jsx```
----

```javascript
import React from 'react';

// react 的 context 组件，可以通过 Provider 在父组件中注册数据，然后所有子组件都能访问到，不需要 props 层层传递，类似java的 LocalThread
const EditableContext = React.createContext();

export default EditableContext;
```

```EditableFormRow.jsx```
----

```javascript
// 替换 Table 的默认行实现组件

import React from 'react';
import { Form } from 'antd';
import EditableContext from './EditableContext';

// 这是一个函数式组件，{ form, index, ...props } 等属性是 Form.create()() 传递过来的
const EditableRow = ({ form, index, ...props }) => (
  // 注册 form，这样后续 Cell 组件可以直接获取这里传入的 form 属性
  <EditableContext.Provider value={form}>
    <tr {...props} />
  </EditableContext.Provider>
);

// 创建 form，并注入 EditableRow
const EditableFormRow = Form.create()(EditableRow);

export default EditableFormRow;
```

```EditableCell.jsx```
----

```javascript
// 替换 Table 默认的单元格实现组件

import React from 'react';
import { Form, Input } from 'antd';
import EditableContext from './EditableContext';

class EditableCell extends React.Component {
  state = {
    editing: false,
  };

  // 点击时切换编辑状态
  toggleEdit = () => {
    this.setState(
      state => ({ editing: !state.editing }),
      () => {
        if (this.state.editing) {
          this.input.focus();
        }
      },
    );
  };

  // 监听 Input 框的输入，并校验值，通过后调用父组件传过来的handleSave方法，将修改返回到父层
  save = e => {
    const { record, handleSave } = this.props;
    this.form.validateFields((error, values) => {
      if (error && error[e.currentTarget.id]) {
        return;
      }
      this.toggleEdit();
      handleSave({ ...record, ...values });
    });
  };

  // 使用context Consumer 取 form 时的函数，返回单元格组件
  renderCell = form => {
    this.form = form;
    const { children, dataIndex, record, title } = this.props;
    const { editing } = this.state;
    // 可编辑则返回 Input，并通过 form 绑定数据，监听事件
    // 不可编辑则返回默认的 Cell 组件
    return editing ? (
      <Form.Item style={{ margin: 0 }}>
        {form.getFieldDecorator(dataIndex, {
          rules: [
            {
              required: true,
              message: `${title} 必输`,
            },
          ],
          initialValue: record[dataIndex],
        })(
          <Input
            ref={node => {
              this.input = node;
              return this.input;
            }}
            onPressEnter={this.save}
            onBlur={this.save}
          />,
        )}
      </Form.Item>
    ) : (
      <div
        className="editable-cell-value-wrap"
        style={{ paddingRight: 24, height: 24, cursor: 'pointer' }}
        onClick={this.toggleEdit}
      >
        {children}
      </div>
    );
  };

  render() {
    const {
      editable,
      dataIndex,
      title,
      record,
      index,
      handleSave,
      children,
      ...restProps
    } = this.props;
    return (
      <td {...restProps}>
        {editable ? (
          // Consumer 的 children 是个函数
          <EditableContext.Consumer>{this.renderCell}</EditableContext.Consumer>
        ) : (
          children
        )}
      </td>
    );
  }
}

export default EditableCell;
```

```index.jsx```
----

```javascript
// 要 export 出的 EditableTable 组件

import React from 'react';
import { Table } from 'antd';
import EditableFormRow from './EditableFormRow';
import EditableCell from './EditableCell';
import './index.less';

class EditableTable extends React.Component {
  render() {
    // 获取父组件传进来的属性，这些属性可以根据需要结合 Table 的原属性进行增加或减少，主要看自己想让哪些受父组件控制
    const { dataSource, columns, handleSave, loading, rowSelection, pagination, total } = this.props;

    // 覆盖 Table 的默认行和单元格实现
    const components = {
      body: {
        row: EditableFormRow,
        cell: EditableCell,
      },
    };

    // 修改 onCell 属性，将监听方法传入 EditableCell 组件
    const columnsMap = columns.map(col => {
      if (!col.editable) {
        return col;
      }
      return {
        ...col,
        onCell: record => ({
          record,
          editable: col.editable,
          dataIndex: col.dataIndex,
          title: col.title,
          handleSave,
        }),
      };
    });

    // 分页组件
    pagination.total = total;
    pagination.showTotal = (count, range) => `显示 ${range[0]}-${range[1]} 共 ${count} 条`;

    return (
      <div>
        <Table
          components={components}
          rowClassName={() => 'editable-row'}
          bordered
          size="small"
          pagination={pagination}
          rowSelection={rowSelection}
          dataSource={dataSource}
          columns={columnsMap}
          loading={loading}
        />
      </div>
    );
  }
}

export default EditableTable;
```

**上面就是 ```EditableTable``` 组件的定义了，只有注入对应的属性就可以实现单元格编辑了，当然这里不包括新增行，删除行和保存部分。**

* 使用 EditableTable 组件

在 pages 文件夹下新建一个页面，在 config 中配置好路由，然后加入以下几个文件

```index.jsx```
----

```javascript
// 页面文件，包括分页，保存，新增，删除

/* eslint-disable no-console */
import React, { Component } from 'react';
import { PageHeaderWrapper } from '@ant-design/pro-layout';
import { Button, Card } from 'antd';
import { connect } from 'dva';
import EditableTable from '@/components/EditableTable';

@connect(({ loading, templateList }) => ({
  dataSource: templateList.data,
  columns: templateList.cols,
  total: templateList.total,
  dataSourceLoading: loading.effects['templateList/fetchServerData'],
}))
class Template extends Component {
  state = {
    selectedRowKeys: [],
    pagination: {
      current: 1,
      pageSize: 50,
      defaultPageSize: 50,
      pageSizeOptions: ['50', '500', '5000'],
      showSizeChanger: true,
      size: 'small',
    },
  };

  componentDidMount() {
    // 挂载完成之后从服务端读取 Table 的列和数据
    const { dispatch } = this.props;
    dispatch({
      type: 'templateList/fetchCols',
    });
    const { current, pageSize } = this.state.pagination;
    dispatch({
      type: 'templateList/fetchServerData',
      payload: {
        page: current,
        pageSize,
      },
    });
  }

  onPageChange = (page, pageSize) => {
    const { pagination } = this.state;
    pagination.current = page;
    this.setState(
      {
        pagination,
      },
      () => {
        const { dispatch } = this.props;
        dispatch({
          type: 'templateList/fetchServerData',
          payload: {
            page,
            pageSize,
          },
        });
      },
    );
  };

  onPageSizeChange = (current, size) => {
    const { pagination } = this.state;
    pagination.current = 1;
    pagination.pageSize = size;
    this.setState(
      {
        pagination,
      },
      () => {
        const { dispatch } = this.props;
        dispatch({
          type: 'templateList/fetchServerData',
          payload: {
            page: 1,
            pageSize: size,
          },
        });
      },
    );
  };

  // Table 的选择框监听
  onSelectChange = selectedRowKeys => {
    this.setState({ selectedRowKeys });
  };

  // 点击删除时，将选中行的 key 传递到服务端进行删除
  handleDelete = () => {
    const { selectedRowKeys } = this.state;
    const { dispatch } = this.props;
    dispatch({
      type: 'templateList/deleteLocalData',
      payload: selectedRowKeys,
    });
    this.setState({
      selectedRowKeys: [],
    });
  };

  // 新增时通过在 dva 中修改数据源，新增一行，然后触发表格重新渲染
  handleAdd = () => {
    const { dispatch } = this.props;
    dispatch({
      type: 'templateList/addLocalRow',
    });
  };

  // 单元格修改完成，保存修改到 dva 中
  handleSave = row => {
    const { dispatch } = this.props;
    dispatch({
      type: 'templateList/updateLocalRow',
      payload: row,
    });
  };

  // 页面点击保存按钮，直接同步 dva 中的全部修改到服务端
  handleServerSave = () => {
    const { dispatch } = this.props;
    dispatch({
      type: 'templateList/syncServerData',
    });
  };

  render() {
    const { dataSource, columns, dataSourceLoading, total } = this.props;
    const { selectedRowKeys, pagination } = this.state;
    pagination.onChange = this.onPageChange;
    pagination.onShowSizeChange = this.onPageSizeChange;

    const rowSelection = {
      selectedRowKeys,
      onChange: this.onSelectChange,
    };

    return (
      <PageHeaderWrapper content="Template">
        <Card bordered>
          <div>
            <Button
              onClick={this.handleAdd}
              type="default"
              style={{ marginBottom: 16, marginRight: 8 }}
            >
              新增
            </Button>
            <Button
              onClick={this.handleDelete}
              type="danger"
              style={{ marginBottom: 16, marginRight: 8 }}
            >
              删除
            </Button>
            <Button
              onClick={this.handleServerSave}
              type="primary"
              style={{ marginBottom: 16, marginRight: 8 }}
            >
              保存
            </Button>
          </div>
          <EditableTable
            dataSource={dataSource}
            columns={columns}
            handleSave={this.handleSave}
            loading={dataSourceLoading}
            rowSelection={rowSelection}
            pagination={pagination}
            total={total}
          />
        </Card>
      </PageHeaderWrapper>
    );
  }
}

export default Template;
```

```model.js```
----

```javascript
// dva model 文件，主要是增删该查的逻辑实现

import { fetchServerData, fetchTmlCols, syncServerData } from './service';

const Model = {
  namespace: 'templateList',

  state: {
    data: [],
    cols: [],
    deleteRowKeys: [],
    total: 0,
  },

  effects: {
    *fetchServerData({ payload }, { call, put }) {
      const response = yield call(fetchServerData, payload);
      yield put({
        type: 'getData',
        payload: response,
      });
    },

    *fetchCols({ payload }, { call, put }) {
      const response = yield call(fetchTmlCols, payload);
      yield put({
        type: 'getCols',
        payload: response,
      });
    },

    *syncServerData(_, { call, put, select }) {
      // 同步数据修改到服务端，通过标记区分新增和更新
      const state = yield select(({ templateList }) => templateList);
      const { data, deleteRowKeys } = state;
      const syncData = data.filter(item => {
        if (item.status && (item.status === 'insert' || item.status === 'update')) {
          return true;
        }
        return false;
      });
      yield call(syncServerData, {
        syncData,
        deleteRowKeys,
      });
      yield put({
        type: 'fetchServerData',
        payload: {
          page: 1,
          pageSize: 50,
        },
      });
    },
  },

  reducers: {
    getData(state, action) {
      return {
        ...state,
        data: action.payload.list,
        total: action.payload.total,
        deleteRowKeys: [],
      };
    },
    getCols(state, action) {
      return {
        ...state,
        cols: action.payload,
      };
    },
    addLocalRow(state) {
      const { data, total } = state;
      const time = new Date().getTime();
      const key = `rowid-${time}`;
      // 新增标记为 insert
      const newItem = { key, status: 'insert' };
      return {
        ...state,
        data: [newItem, ...data],
        total: total + 1,
      };
    },
    updateLocalRow(state, action) {
      const row = action.payload;
      const newData = [...state.data];
      const index = newData.findIndex(item => row.key === item.key);
      const item = newData[index];
      // 如果是服务端的数据则打上 update 标记
      if (!item.status || item.status !== 'insert') {
        item.status = 'update';
      }
      newData.splice(index, 1, {
        ...item,
        ...row,
      });
      return {
        ...state,
        data: newData,
      };
    },
    deleteLocalData(state, action) {
      const selectedRowKeys = action.payload;
      const { data, deleteRowKeys, total } = state;
      deleteRowKeys.push(...selectedRowKeys);
      const newData = [];
      data.forEach(element => {
        if (!selectedRowKeys.includes(element.key)) {
          newData.push(element);
        }
      });
      return {
        ...state,
        data: newData,
        total: total - deleteRowKeys.length,
      };
    },
  },
};

export default Model;
```

```service.js```
----

```javascript
// api请求文件

import request from '@/utils/request';

export async function fetchServerData(params) {
  return request('/api/fetch_server_data', {
    params,
  });
}

export async function syncServerData(params) {
  return request('/api/sync_server_data', {
    method: 'POST',
    data: params,
  });
}

export async function fetchTmlCols() {
  return request('/api/tmlcols');
}
```

```_mock.js```
----

```javascript
// mock 文件，模拟服务端数据的增删改查

const name = ['163', 'netease', 'com', '西湖', '湘湖', 'ant design pro', '千岛湖', 'Test Name'];
const address = [
  '西湖区湖底公园1号',
  '滨江区网商路599号',
  '浙江省杭州市滨江区网商路699号',
  '北京市亦庄经济开发区',
  '上海市青浦区汇联路xxx号',
  '长河路铂金名筑小区',
];

let initData = [];
for (let i = 1; i <= 12345; i += 1) {
  initData.push({
    key: `row-${i}`,
    name: name[i % 8],
    age: i,
    cardId: 431121,
    phone: 1827311,
    address: address[i % 6],
  });
}

function getServerData(req, res) {
  const params = req.query;
  const { page, pageSize } = params;
  const start = (page - 1) * pageSize;
  const end = page * pageSize - 1 > initData.length ? initData.length - 1 : page * pageSize - 1;

  const list = [];

  for (let i = start; i <= end; i += 1) {
    list.push(initData[i]);
  }
  return res.json({
    list,
    total: initData.length,
  });
}

function syncServerData(req) {
  const { syncData, deleteRowKeys } = req.body;
  initData = initData.filter(item => !deleteRowKeys.includes(item.key));

  syncData.forEach(item => {
    if (item.status === 'insert') {
      initData.push({ ...item, status: '' });
    } else {
      const index = initData.findIndex(it => it.key === item.key);
      const findItem = initData[index];
      initData.splice(index, 1, {
        ...findItem,
        ...item,
        status: '',
      });
    }
  });
}

export default {
  'GET  /api/fetch_server_data': getServerData,

  'POST /api/sync_server_data': syncServerData,

  'GET /api/tmlcols': [
    {
      title: '姓名',
      dataIndex: 'name',
      key: 'name',
      width: 300,
      editable: true,
    },
    {
      title: '年龄',
      dataIndex: 'age',
      key: 'age',
      width: 300,
      editable: true,
    },
    {
      title: '住址',
      dataIndex: 'address',
      key: 'address',
      width: 300,
      editable: true,
    },
    {
      title: '电话',
      dataIndex: 'phone',
      key: 'phone',
      editable: true,
    },
  ],
};
```

总结
---

以上就是实现一个单元格编辑，增删改查表格的全部代码了，发现使用 react 技术栈大大降低了后端程序员实现前端组件的难度，这要是用 jquery 框架来实现，那肯定要复杂很多很多了。但是也有个不好的地方，那就是性能跟 juqery 框架实现的差距有点大，如果把分页设为5000条，翻页和编辑的卡顿就非常明显了，这跟 react 的机制有关系，当然这跟我粗略实现，没有考虑性能优化也有很大关系，随便操作都会导致页面重新渲染，后续实际应用时可以考虑优化优化。