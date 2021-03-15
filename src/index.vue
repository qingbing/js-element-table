<template>
  <el-table
    :data="tableData"
    :style="{ width: tableWidth }"
    :stripe="stripe"
    :border="border"
    :row-class-name="rowClass"
    :height="tableHeight"
    :max-height="maxHeight"
    :empty-text="emptyText"
    :tooltip-effect="tooltipEffect"
  >
    <template v-for="item in headers">
      <!-- 当表头设置了 component 时，在 scope 中使用 component 的形式，组件名为 item.component 设置的值 -->
      <el-table-column
        v-if="item.component"
        :key="'' + uniqid + item.name"
        :prop="item.name"
        :label="item.label"
        :width="isEmpty(item.width) ? 'auto' : item.width"
        :fixed="item.fixed ? item.fixed : false"
        :align="item.align ? item.align : 'center'"
        :show-overflow-tooltip="item.tooltip ? item.tooltip : false"
        :resizable="item.resizable ? item.resizable : false"
      >
        <template slot-scope="scope">
          <component
            :is="item.component"
            :scope="scope"
            :index="scope.$index"
            :property="scope.column.property"
            :row="scope.row"
          ></component>
        </template>
      </el-table-column>
      <!-- 常规的显示 -->
      <el-table-column
        v-else
        :key="'' + uniqid + item.name"
        :prop="item.name"
        :label="item.label"
        :width="isEmpty(item.width) ? 'auto' : item.width"
        :fixed="item.fixed ? item.fixed : false"
        :align="item.align ? item.align : 'center'"
        :show-overflow-tooltip="item.tooltip ? item.tooltip : false"
        :resizable="item.resizable ? item.resizable : false"
      ></el-table-column>
    </template>
  </el-table>
</template>

<script>
// 导入包
import { isEmpty, isFunction, uniqid, each, dump } from "@qingbing/helper";

// 导出
export default {
  props: {
    /**
     * 表格配置
     */
    // table 宽度
    tableWidth: {
      type: String,
      default: "100%",
    },
    // table 是否带斑马纹
    stripe: {
      type: Boolean,
      default: true,
    },
    // table 是否需要竖直方向的边框
    border: {
      type: Boolean,
      default: true,
    },
    // table 是否需要竖直方向的边框
    rowClass: {
      type: Function,
      default: () => {
        return ""; // red
      },
    },
    // table 是否需要竖直方向的边框
    tableHeight: {
      type: String,
      default: undefined,
    },
    // table 是否需要竖直方向的边框
    maxHeight: {
      type: String,
      default: undefined,
    },
    // 空数据时显示的文本内容
    emptyText: {
      type: String,
      default: "暂无数据",
    },
    // tooltip effect 属性，dark/light
    tooltipEffect: {
      type: String,
      default: "light",
    },
    /**
     * 数据相关配置
     */
    // 组件唯一标志符
    uniqid: {
      type: String,
      default: () => {
        return uniqid();
      },
    },
    // 标题栏
    getHeaders: {
      type: Function,
      required: true,
    },
    // 标题栏
    getTableData: {
      type: Function,
      required: true,
    },
    // 数据渲染前的处理函数
    beforeRender: {
      type: Function,
    },
    pagination: {
      type: Boolean,
      default: false,
    },
    pageNo: {
      type: Number,
      default: 1,
    },
    pageSize: {
      type: Number,
      default: 10,
    },
  },
  data() {
    this.getHeaders((res) => {
      // 计算并保存真正的 header
      const headers = {};
      each(res, (re, idx) => {
        if (isEmpty(re.name) || isEmpty(re.label)) {
          dump.error(`表格组件header头 ${idx} 必须指定字段和字段名`);
        }
        if (!isEmpty(re.component)) {
          const com = this.$parent.$options.components[re.component];
          if (isEmpty(com)) {
            dump.error(`父组件无组件 "${re.component}" 可继承`);
          }
          this.$options.components[re.component] = com;
        }
        headers[re.name];
      });
      this.headers = res;
      // header 处理完后处理初始化数据
      this.getTableData((res) => {
        this.addIdx(res);
        this.tableData = res;
      });
    });
    return {
      headers: {},
      tableData: [],
    };
  },
  methods: {
    isEmpty,
    addIdx(res) {
      const firstIdx = this.pagination ? (this.pageNo - 1) * this.pageSize : 0;
      const hasBeforeRender = isFunction(this.beforeRender);
      each(res, (item, idx) => {
        item._idx = firstIdx + parseInt(idx) + 1;
        if (hasBeforeRender) {
          this.beforeRender(item, idx, res);
        }
      });
    },
  },
};
</script>