<template>
  <div>
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
              :params="item.params"
              :editable="editable"
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

    <section class="page-wrapper" v-if="!isEmpty(pagination)">
      <el-pagination
        :layout="paginationLayout"
        :total="pagination.total"
        :hide-on-single-page="true"
        @current-change="pageChange"
        @size-change="sizeChange"
      >
      </el-pagination>
    </section>
  </div>
</template>

<script>
// 导入包
import {
  isEmpty,
  isUndefined,
  isObject,
  isFunction,
  uniqid,
  each,
  dump,
} from "@qingbing/helper";

// 导出
export default {
  props: {
    /**
     * 表格配置
     */
    // table 是否带斑马纹
    stripe: {
      type: Boolean,
      default: true,
    },
    // table 边框表格
    border: {
      type: Boolean,
      default: true,
    },
    // table 行添加 class
    rowClass: {
      type: Function,
      default: () => "",
    },
    // table 宽度
    tableWidth: {
      type: String,
      default: undefined,
    },
    // table 高度
    tableHeight: {
      type: String,
      default: undefined,
    },
    // table 最大高度
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
      default: () => uniqid(),
    },
    // 数据渲染前的处理函数
    beforeRender: {
      type: Function,
    },
    // 标题栏
    getHeaders: {
      type: Function,
      required: true,
    },
    // 获取数据
    getTableData: {
      type: Function,
      required: true,
    },
    // 分页信息
    pagination: {
      type: Object,
      default: undefined,
    },
    // 分页组件布局
    paginationLayout: {
      type: String,
      default: "total,prev,pager,next,jumper",
    },
    /**
     * 组件配置
     */
    // 可编辑，需要结合组件 @qingbing/element-cell-edit 使用
    editable: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    // 分页参数规范
    if (isObject(this.pagination)) {
      this.pagination.pageNo = this.pagination.pageNo ?? 1;
      this.pagination.pageSize = this.pagination.pageSize ?? 10;
      if (this.pagination.pageNo < 1) {
        this.pagination.pageNo = 1;
      }
      if (this.pagination.pageSize < 1) {
        this.pagination.pageSize = 1;
      }
      if (this.pagination.total < 0) {
        this.pagination.total = 0;
      }
    }
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
      // header 处理完后刷新数据列表
      this.refreshTable();
    });
    const R = {};
    if (isUndefined(this.headers)) {
      R.headers = {};
    }
    if (isUndefined(this.tableData)) {
      R.tableData = [];
    }
    return R;
  },
  methods: {
    isEmpty,
    // 为数据添加序号
    addIdx(res) {
      let firstIdx = 0;
      if (isObject(this.pagination)) {
        firstIdx = (this.pagination.pageNo - 1) * this.pagination.pageSize;
      }
      const hasBeforeRender = isFunction(this.beforeRender);
      each(res, (item, idx) => {
        item._idx = firstIdx + parseInt(idx) + 1;
        if (hasBeforeRender) {
          this.beforeRender(item, idx, res);
        }
      });
    },
    // 重刷 table 数据
    refreshTable() {
      this.getTableData((res) => {
        let flag = true;
        if (isObject(this.pagination)) {
          if (isUndefined(res.total)) {
            flag = false;
            dump.error("分页返回结果中必须包含总条数 total");
          } else {
            this.pagination.total = parseInt(res.total);
            res = res.data;
          }
        }
        if (flag) {
          this.addIdx(res);
          this.tableData = res;
        }
      });
    },
    // 改变了 pageNo
    pageChange(curPage) {
      this.pagination.pageNo = curPage;
      this.refreshTable();
    },
    // 改变了 pageSize
    sizeChange(curSize) {
      this.pagination.pageSize = curSize;
      this.refreshTable();
    },
  },
};
</script>
<style scoped>
.page-wrapper {
  padding: 10px 20px;
}
</style>