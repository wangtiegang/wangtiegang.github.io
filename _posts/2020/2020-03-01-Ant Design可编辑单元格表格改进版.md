---
layout: post
title: "Ant Design可编辑单元格表格改进版"
subtitle: "基于ant design@4 版本"
date: 2020-03-01 19:05:00
author: "wangtiegang"
header-img: ""
catalog: false
tags:
  - React
  - antd
  - JavaScript
---

之前在空闲的时候按照 antd 官方的可编辑单元格的示例改进实现过一个 [demo](https://wangtiegang.github.io/2019/11/30/ant-design-pro%E5%AE%9E%E7%8E%B0%E5%8F%AF%E7%BC%96%E8%BE%91%E5%8D%95%E5%85%83%E6%A0%BCTable/) ，然后马马虎虎能用，自己也弄明白了实现的原理。然后没有实际的项目，就没再继续改进了。

最近真的要用 antd pro 做项目了，突然发现几个月不看，已经变成 hook 风格了，前端变化也太快了，有点看不懂，又花了些时间学习才找回来感觉。。然后在实际项目中需要可编辑单元格，又去 ant 官网看最新 @4 版本的示例，弄明白之后，结合 hook 进行改造增强，实现了一个比上次好一丢丢的版本，虽然还存在很多可以改进的地方，但是项目进度赶，只能先出成果，后续再进行改进了。

### 实现

此次的可编辑表格具有以下特点

- 采用函数式组件和 hook
- 使用了最新 ant@4 版本的 Form 特性
- 支持配置单元格编辑组件类型，目前支持字符串，数字，日期，下拉框，可根据需要扩展

#### EditableTable 代码

```javascript
import React, { useContext, useState, useEffect, useRef } from "react";
import { Table, Input, Form, Select, InputNumber } from "antd";
import styles from "./index.less";
import StringDatePicker from "../StringDatePicker";

const EditableContext = React.createContext();

const EditableRow = ({ index, ...props }) => {
  const [form] = Form.useForm();
  return (
    <Form form={form} component={false}>
      <EditableContext.Provider value={form}>
        <tr {...props} />
      </EditableContext.Provider>
    </Form>
  );
};

const EditableCell = ({
  title,
  editable,
  editor,
  children,
  dataIndex,
  rules,
  record,
  handleSave,
  ...restProps
}) => {
  const [editing, setEditing] = useState(false);
  const inputRef = useRef();
  const form = useContext(EditableContext);
  useEffect(() => {
    if (editing) {
      if (inputRef.current) {
        inputRef.current.focus();
      }
    }
  }, [editing]);

  const toggleEdit = () => {
    setEditing(!editing);
    form.setFieldsValue({
      [dataIndex]: record[dataIndex]
    });
  };

  const save = async () => {
    try {
      const values = await form.validateFields();
      toggleEdit();
      handleSave({ ...record, ...values });
    } catch (errInfo) {
      // eslint-disable-next-line no-console
      console.log("行保存失败:", errInfo);
    }
  };

  let childNode = children;

  if (editable) {
    // 根据传递过来的 column 属性判断使用哪种输入组件
    let editorNode;
    if (editor.type === "string") {
      editorNode = (
        <Input
          ref={inputRef}
          style={{
            width: "100%"
          }}
          onPressEnter={save}
          onBlur={save}
        />
      );
    } else if (editor.type === "number") {
      editorNode = (
        <InputNumber
          style={{
            width: "100%"
          }}
          autoFocus
          onPressEnter={save}
          onBlur={save}
        />
      );
    } else if (editor.type === "dropdown") {
      editorNode = (
        <Select
          style={{
            width: "100%"
          }}
          autoFocus
          showSearch
          optionFilterProp="label"
          options={editor.options}
          onSelect={save}
          onBlur={save}
        />
      );
    } else if (editor.type === "date") {
      editorNode = (
        // StringDatePicker 是自定义的日期选择组件，直接返回字符串类型的值
        <StringDatePicker
          style={{
            width: "100%"
          }}
          format="YYYY/MM/DD"
          autoFocus
          onStringChange={save}
          onBlur={save}
        />
      );
    } else {
      editorNode = "";
    }

    childNode = editing ? (
      <Form.Item
        style={{
          margin: 0
        }}
        name={dataIndex}
        rules={rules}
      >
        {editorNode}
      </Form.Item>
    ) : (
      <div
        className={styles.editableCellValueWrap}
        style={{
          paddingRight: 24
        }}
        onClick={toggleEdit}
      >
        {children}
      </div>
    );
  }

  return <td {...restProps}>{childNode}</td>;
};

const EditableTable = props => {
  const { columns, handleSave } = props;

  // 通过onCell设置单元格属性
  const newColumns = columns.map(col => {
    if (!col.editable) {
      return col;
    }

    return {
      ...col,
      onCell: record => ({
        record,
        editable: col.editable,
        editor: col.editor,
        dataIndex: col.dataIndex,
        title: col.title,
        rules: col.rules,
        handleSave
      })
    };
  });

  // 替换原生的row和cell组件
  const components = {
    body: {
      row: EditableRow,
      cell: EditableCell
    }
  };

  const newProps = { ...props, columns: newColumns, components };

  return <Table {...newProps} rowClassName={styles.editableRow} />;
};

export default EditableTable;
```

样式表

```css
.editable-cell {
  position: relative;
}

.editableCellValueWrap {
  padding: 5px 12px;
  cursor: pointer;
}

.editableRow:hover .editableCellValueWrap {
  min-height: 32px;
  padding: 4px 11px;
  border: 1px solid #d9d9d9;
  border-radius: 4px;
}

[data-theme="dark"] .editableRow:hover .editableCellValueWrap {
  min-height: 32px;
  border: 1px solid #434343;
}
```

### 使用方式

由于没有使用 `TypeScript` ，不好继承和对属性进行限制，所以在使用的时候必须自己注意传递必须的属性，除了 `columns` 属性变化了，需要更多信息外，其他属性跟原生的 `Table` 组件一致

```javascript
const TestComponent = () => {
  // 新增时赋值 id
  const [count, setCount] = useState(1);
  // 表格数据
  const [dataSource, setDataSource] = useState();

  // 新增行
  const handleAdd = () => {
    const newData = {
      id: `_add_${count}`,
      __status: "add"
    };
    setDataSource([newData, ...dataSource]);
    setCount(count + 1);
  };
  
  // 列的属性增加了 editable，editor，rules
  const columns = [
    {
      ellipsis: true,
      title: "公司名称",
      dataIndex: "companyName",
      width: "300px",
      editable: true, // 是否可编辑
      editor: {
        type: "string" // 支持 string，number，dorpdown，date，可自行扩展
      }, // 当可编辑时，必须提供 editor 属性
      rules: [
        {
          required: true,
          message: "公司名称必填"
        }
      ] // 用来编辑时校验，同 form rules配置
    },
    {
      ellipsis: true,
      title: "公司编码",
      dataIndex: "companyCode",
      width: "300px",
      editable: true,
      editor: {
        type: "string"
      }
    }
  ];

  // 单元格编辑后的保存函数
  const handleRowSave = row => {
    const newRow = row;
    if (row.__status !== "add") {
      newRow.__status = "update";
    }
    const newDataSource = [...dataSource];
    const index = newDataSource.findIndex(item => newRow.id === item.id);
    const item = newDataSource[index];
    newDataSource.splice(index, 1, { ...item, ...newRow });
    setDataSource(newDataSource);
  };

  return (
    // EditableTable 可以接受所有 Table 组件的属性
    <EditableTable
      size="small"
      rowKey="id"
      columns={columns}
      dataSource={dataSource}
      handleSave={handleRowSave}
      rowSelection={rowSelection}
      pagination={pagination}
    />
  );
};

export default TestComponent;
```

以上就是全部了，使用代码只是一个很简单粗糙的示例，实际使用中需要定义增删改查方法，结合 hook 就可实现从数据库读取数据进行操作了，同时也可以定义表格的翻页筛选组件，这个就跟原生的 ```Table``` 使用方式一样了。

可编辑单元格表格在实际项目中使用非常频繁，后续可以把增删改查封装进去，使用的时候只传递一组 url 和 dataSource 的 Fields 就好了，这样可以简化很多工作量，不过现阶段水平时间有限，只能更熟练后进行重构了
