<template>
  <div class="record">
    <!-- 全部股票池 -->
    <el-form
      :model="form"
      ref="form"
      label-width="82px"
      :inline="true"
      size="mini"
    >
      <el-form-item label="创建时间" prop="tiems">
        <el-date-picker
          v-model="form.tiems"
          value-format="yyyy-MM-dd"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          @change="pickChange"
        >
        </el-date-picker>
      </el-form-item>
      <!-- <el-form-item label="入池后收益" prop="profit">
        <el-select
          v-model="form.profit"
          placeholder="请选择"
          @change="selectChange"
          clearable
        >
          <el-option
            v-for="item in optionsProfit"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item> -->
      <el-form-item label="来源" prop="sourceModel">
        <el-input
          v-model="form.sourceModel"
          @keyup.enter.native="sourceChange"
          @input="sourceChange"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item label="审核人" prop="reviewerModel">
        <el-input
          clearable
          v-model="form.reviewerModel"
          @keyup.enter.native="reviewedChange"
          @input="reviewedChange"
        ></el-input>
      </el-form-item>
    </el-form>
    <div class="seachInput">
      <div></div>
      <el-input
        clearable
        v-model="titlesearch"
        placeholder=" 搜索代码或名称"
        style="width: 200px;"
        class="filter-item"
        suffix-icon="el-icon-search"
        @keyup.enter.native="handleFilter"
        @input="handleFilter"
      />
    </div>
    <tables
      :tableData="tableData1"
      :cloumns="cloumns"
      :headerSelection="false"
      :indexType="true"
      :iconType="false"
    >
      <el-table-column prop="srcType" label="来源" align="center">
        <template slot-scope="scope">
          {{ scope.row.srcType === '1' ? '指数' : '用户' }}
        </template>
      </el-table-column>
    </tables>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
  </div>
</template>
<script>
import Pagination from '@/components/Pagination'
import tables from '@/views/fundPool/components/table'
import { getStockPoolPage } from '@/api/productStockPool/productStockPool'
export default {
  name: 'stockPoolAllShares',
  components: {
    Pagination,
    tables
  },
  data () {
    return {
      titlesearch: '',
      value1: '',
      form: {
        tiems: '',
        profit: '1',
        sourceModel: '',
        reviewerModel: ''
      },
      tableData1: [],
      cloumns: [
        {
          prop: 'stkcode',
          lable: '股票代码'
        },
        {
          prop: 'stkname',
          lable: '股票名称'
        },

        {
          prop: 'assetsType',
          lable: '所属资产类别'
        },
        {
          prop: 'poolDate',
          lable: '入池时间'
        },
        {
          prop: 'productProfit',
          lable: '入池后收益'
        },
        {
          prop: 'inPrice',
          lable: '推荐理由'
        },
        {
          prop: 'examineBy',
          lable: '审核'
        },

        {
          prop: 'createBy',
          lable: '投顾'
        }
      ],
      listQuery: {
        page: 1,
        limit: 10
      },
      total: 20,
      optionsProfit: [
        {
          value: '1',
          label: '亏损'
        },
        {
          value: '2',
          label: '0%-20%'
        },
        {
          value: '3',
          label: '20%-50%'
        },
        {
          value: '4',
          label: '50%以上'
        }
      ]
    }
  },
  created () {
    this.getStockPoolPageFun()
  },
  watch: {
    'form.tiems' (newV, oldV) {
      if (newV == null) {
        this.form.tiems = []
      }
    }
  },

  methods: {
    getList () {
      this.getStockPoolPageFun()
    },
    getStockPoolPageFun () {
      getStockPoolPage({
        key: this.titlesearch,
        page: this.listQuery.page,
        limit: this.listQuery.limit,
        offset: 0,
        startProductProfit: '', //	开始上架后收益
        endProductProfit: '', //	结束上架后收益
        examineBy: this.form.reviewerModel,
        startDate: this.form.tiems[0],
        endDate: this.form.tiems[1]
      }).then(res => {
        console.log(res)
        this.tableData1 = res.data.data.list
        this.total = res.data.data.total
      })
    },
    iconClick () {
      console.log(1)
    },
    handleFilter () {
      this.getStockPoolPageFun()
    },
    selectChange (val) {
      console.log(val)
      this.getStockPoolPageFun()
    },
    pickChange (tiem) {
      if (tiem === null) {
        this.form.tiems = ''
      }
      this.getStockPoolPageFun()
    },
    sourceChange () {
      console.log(this.form.sourceModel)
      this.getStockPoolPageFun()
    },
    reviewedChange () {
      console.log(this.form.reviewerModel)
      this.getStockPoolPageFun()
    }
  }
}
</script>
<style lang="less" scoped>
.record {
  margin-top: 10px;
}
.seachInput {
  display: flex;
  justify-content: space-between;
}
</style>
