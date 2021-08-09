<template>
  <div class="table">
    <el-table
      :data="tableData"
      stripe
      style="width: 100%"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="50" v-if="headerSelection">
      </el-table-column>
      <el-table-column type="index" width="50" v-if="indexType" label="序号">
      </el-table-column>
      <el-table-column width="50" v-if="iconType" label="状态" align="center">
        <template slot-scope="scope">
          <i
            class="el-icon-warning-outline red"
            v-if="scope.row.fwmsgShowstate === '2'"
            @click="iconClick(scope)"
          ></i>
          <i
            class="el-icon-warning-outline yelow"
            v-if="scope.row.fwmsgShowstate === '1'"
            @click="iconClick(scope)"
          ></i>
          <i
            class="el-icon-warning-outline blur"
            v-if="scope.row.fwmsgShowstate === '0'"
            @click="iconClick(scope)"
          ></i>
        </template>
      </el-table-column>
      <el-table-column
        v-for="item in cloumns"
        :key="item.prop"
        :prop="item.prop"
        :index="item.index"
        :label="item.lable"
        align="center"
        :sortable="item.sortableState"
        :formatter="if_null"
      >
      </el-table-column>

      <el-table-column
        fixed="right"
        label="操作"
        align="center"
        v-if="options.length"
      >
        <!-- v-if="options.length" -->
        <template slot-scope="scope">
          <div>
            <slot name="option" :row="scope.row"></slot>
          </div>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
<script>
export default {
  props: {
    tableData: {
      type: Array,
      default: () => []
    },
    cloumns: {
      type: Array,
      default: () => []
    },
    options: {
      type: Array,
      default: () => []
    },
    headerSelection: {
      type: Boolean,
      default: false
    },
    indexType: {
      type: Boolean,
      default: false
    },
    iconType: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {}
  },
  methods: {
    if_null (row, column, cellValue) {
      return cellValue || '--'
    },
    handleClick (row, option) {
      option(row)
    },
    handleSelectionChange (val) {
      console.log(val)
      this.$emit('getCheckList', val)
    },
    iconClick (scope) {
      this.$emit('iconClick', scope)
    }
  }
}
</script>
<style lang="less">
.red {
  color: red;
}
.yelow {
  color: rgb(194, 52, 0);
}
.blur {
  color: rgb(40, 209, 68);
}
/deep/ .btnColor {
  color: #ccc !important;
}
</style>
