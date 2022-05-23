# 封装 vue-element-ui 的组件
二次封装 element-ui 的table组件，使其使用更方便

### ====== 版本说明 ======
- 1.0.0 : 代码初始化
- 1.0.1 : 完善备注
  - 添加 editable 属性，控制是否可编辑
- 1.0.2
  - 添加关于对于字段 default 的设置使用，只用当返回字段 undefined 时有效
- 1.0.3
  - 解决分页条数传递问题
- 1.0.4
  - 在 header 头中加入 options 为对象或数组时，可以自动转换成对应 label
- 1.0.5
  - 表格中加入了 cell-table 组件，可以使得表格能够直接编辑，前提是需要设置好 header 的params 数据
- 1.0.6
  - 返回表头支持对象，为对象时，可以不用设置field字段
- 1.0.7
  - 增加数据未空时，也直接显示表头设置的默认值
- 1.0.8
  - 将 beforeRender 做成了 $emit 消息传递
- 1.0.9
  - 扩展了图片列表功能

## 1. 安装
```
npm install @qingbing/element-table
```

## 2. 传递参数

### 2.1 prop 参数

| 参数名 | 参数类型 | 数据类型 | 必填 | 默认 | 描述 |
|:---|:---|:---|:---|:---|:---|
| getHeaders | 属性 | Function | 是 | - | 获取标题栏 |
| getTableData | 属性 | Function | 是 | - | 获取数据 |
| stripe | 属性 | Boolean | 否 | true | table 是否带斑马纹 |
| border | 属性 | Boolean | 否 | true | table 边框表格 |
| rowClass | 属性 | Function | 否 | "" | table 行添加 class |
| tableWidth | 属性 | String | 否 | undefined | table 宽度 |
| tableHeight | 属性 | String | 否 | undefined | table 高度 |
| maxHeight | 属性 | String | 否 | undefined | table 最大高度 |
| emptyText | 属性 | String | 否 | 暂无数据 | 空数据时显示的文本内容 |
| tooltipEffect | 属性 | String | 否 | light | tooltip effect 属性，dark/light |
| uniqid | 属性 | String | 否 | uniqid() | 组件唯一标志符 |
| pagination | 属性 | Object | 否 | undefined | 分页信息 |
| paginationLayout | 属性 | String | 否 | total,prev,pager,next,jumper | 分页组件布局 |
| editable | 属性 | Boolean | 否 | true | 可编辑，需要结合组件 @qingbing/element-cell-edit 使用 |

### 2.2 $emit 参数

| 参数名 | 参数类型 | 数据类型 | 必填 | 默认 | 描述 |
|:---|:---|:---|:---|:---|:---|
| beforeRender | 属性 | Function | 否 | - | 数据渲染前的处理函数 |



## 3. 使用示例
```vue
<template>
  <div>
    <el-divider>表格编辑</el-divider>
    <e-table
      :getHeaders="getHeaders"
      :getTableData="getData"
      :beforeRender="beforeRender"
    ></e-table>
  </div>
</template>

<script>
// 导入包
import { merge } from "@qingbing/helper";
import { ajaxMethod } from "./../../utils/unit";

export default {
  components: {
    ETable: () => import("@qingbing/element-table"),
    operate: () => import("./../../components/operate"),
    celledit: () => import("@qingbing/element-cell-edit"),
  },
  data() {
    return {
      beforeRender(item, idx) {},
    };
  },
  methods: {
    getHeaders(cb) {
      const editParams = {
        dataType: "user",
        saveUrl: "/user-save",
        saveCallback(cb, change, properties, params) {
          ajaxMethod(
            "user-cell-edit",
            merge({ id: properties.id }, change),
            "post",
            (res) => cb(res)
          );
        },
      };
      cb([
        { field: "_idx", label: "序号", fixed: "left" },
        { field: "date", label: "日期", width: "100", default: "0000-00-00" },
        {
          field: "is_open",
          label: "是否开放",
          //   width: "100",
          component: "celledit",
          params: merge(editParams, {
            type: "switch",
          }),
        },
        {
          field: "username",
          label: "用户名",
          component: "celledit",
          params: merge(editParams, {
            type: "text",
          }),
        },
        {
          field: "field",
          label: "姓名",
          width: "150",
          component: "celledit",
          params: merge(editParams, {
            type: "textarea",
          }),
        },
        {
          name: "age",
          label: "年龄",
          width: "80",
          component: "celledit",
          params: merge(editParams, {
            type: "number",
          }),
        },
        {
          name: "sex",
          label: "性别",
          width: "80",
          component: "celledit",
          params: merge(editParams, {
            type: "select",
            options: {
              0: "密",
              1: "男",
              2: "女",
            },
          }),
        },
        {
          name: "operate",
          label: "操作",
          component: "operate",
          params: {
            addUrl: "/user-add",
            editUrl: "/user-edit",
            viewUrl: "/user-view",
          },
        },
      ]);
    },
    getData(cb) {
      cb([
        {
          id: "1",
          username: "RzW",
          is_open: 1,
          name: "万磊",
          sex: "nan",
          age: "22",
          date: "2011-06-18",
          info: "电的更看事众心中响求型可适千情。",
        },
        {
          id: "2",
          username: "s3Y",
          is_open: "0",
          name: "乔洋",
          sex: "nv",
          age: "24",
          date: "1987-09-06",
          info: "置个京等拉己指林众通外手变意老。",
        },
        {
          id: "3",
          username: "mc3h#F",
          name: "孔伟",
          sex: "2",
          age: "18",
          date: "2009-03-22",
          info: "影民离世文为任置格资支放你高京。",
        },
      ]);
    },
  },
};
</script>
```

