---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: antd  TreeSelect获取父节点的值
slug: antd
featured: false
draft: false
tags:
  - antd
description: antd TreeSelect获取父节点的值
---

在antd对Treeselect组件的渲染中，onChange事件是无法获取父元素的值的，官方解释是处于对性能的考虑，没有对父元素进行关联。

文档末尾也给出了如何获取父元素值的方法，解题思路是：根据treeData的数据结构利用递归回溯去查找父节点的值

忠于文档～

```javascript
import React from "react";
import { TreeSelect } from "antd";
import { Post } from "../../api/index";

const valueMap = {};
function loops(list, parent) {
  return (list || []).map(({ children, value }) => {
    const node = (valueMap[value] = {
      parent,
      value,
    });
    node.children = loops(children, node);
    return node;
  });
}
// 查找父节点的值
function getPath(value) {
  const path = [];
  let current = valueMap[value];
  while (current) {
    path.unshift(current.value);
    current = current.parent;
  }
  return path;
}

/**
 * 格式化树形机构
 */
function formatTree(
  list = [],
  formatFun,
  childrenName = "children",
  index = 0,
  dep = -1
) {
  return list.map((z, i) => {
    const hasChildren = !!(z[childrenName] || []).length;
    let c = null;
    if (dep === -1 || index < dep) {
      c = hasChildren
        ? formatTree(
            z[childrenName] || [],
            formatFun,
            childrenName,
            index + 1,
            dep
          )
        : null;
      console.log(111, z.name, index < dep, c);
    }
    //c = hasChildren ? formatTree(z[childrenName] || [], formatFun, childrenName, index + 1) : null;
    return {
      ...formatFun(z, i),
      hasChildren,
      level: index,
      children: c,
    };
  });
}

class Test extends React.PureComponent {
  state = {
    districts: "北京",
    districtsList: [],
  };

  componentDidMount() {
    Post("district/treeList").then(res => {
      const districts = res.data[0].name;
      getData.setDistricts(districts); // 保存为全局
      this.setState(
        {
          districtsList: formatTree(
            res.data,
            function (z) {
              return {
                title: z.name,
                value: z.name,
              };
            },
            "childList",
            0,
            1
          ),
          districts: districts,
        },
        () => {
          // 这是我自己其他的业务逻辑 可忽略
          this.update();
          const updateTime = process.env.REACT_APP_SITE_REFRESH_TIME;
          this.timer = setInterval(() => {
            this.update();
          }, +updateTime * 1000);
          // 这才是重点
          loops(this.state.districtsList);
        }
      );
    });
  }

  onChangeArea = value => {
    console.log(getPath(value));
    const new_district = getPath(value).join("-");
    getData.setDistricts(new_district);
    this.setState({
      districts: value,
    });
  };

  render() {
    const { districts, districtsList } = this.state;
    return (
      <TreeSelect
        showSearch
        dropdownMatchSelectWidth={300}
        size="small"
        value={districts}
        treeData={districtsList}
        placeholder="请选择区域"
        treeDefaultExpandAll
        onChange={this.onChangeArea}
      />
    );
  }
}
```
