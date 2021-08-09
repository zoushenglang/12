<template>
  <div class="notGrounding">
    <!-- 未上架股票池 -->

    <el-form
      size="mini"
      :model="form"
      ref="form"
      label-width="82px"
      :inline="true"
      style="margin-top:20px"
    >
      <el-form-item label="上下架状态" prop="stockStatus">
        <el-select
          v-model="form.stockStatus"
          placeholder="请选择"
          @change="selectChange"
          clearable
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="创建时间" prop="tiems">
        <el-date-picker
          v-model="tiemValue"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          @change="pickerChange"
          value-format="yyyy-MM-dd"
        >
        </el-date-picker>
      </el-form-item>
      <div class="seachInput">
        <div>
          <el-button type="primary" size="mini" @click="onceImport"
            >单个导入</el-button
          >
          <el-button type="primary" size="mini" @click="groundingsAll">
            批量导入</el-button
          >
          <!-- @click="groundings" -->
          <el-button type="primary" size="mini" @click="handleBatchShelves">
            批量申请上架</el-button
          >
          <el-button
            type="primary"
            size="mini"
            @click="batchWithdraPutOnApplyStockAll"
          >
            批量撤回申请</el-button
          >
          <el-button type="primary" size="mini" @click="batchDeleteStockAll">
            批量删除</el-button
          >
        </div>
        <el-input
          v-model="titlesearch"
          placeholder=" 搜索代码或名称"
          style="width: 200px;"
          class="filter-item"
          suffix-icon="el-icon-search"
          @keyup.enter.native="handleFilter"
          @input="handleFilter"
          clearable
        />
      </div>
    </el-form>
    <tables
      :tableData="tableData1"
      :cloumns="cloumns"
      :options="optionsTable"
      :headerSelection="true"
      @iconClick="fIconClick"
      @getCheckList="FgetCheckList"
      ref="multipleTable"
    >
      <template v-slot:option="slotProps">
        <el-button
          type="text"
          @click="() => watchBtn(slotProps.row)"
          :class="
            slotProps.row.stockStatus == '0' ||
            slotProps.row.stockStatus == '5' ||
            slotProps.row.stockStatus == '4'
              ? ''
              : 'btnColor'
          "
          >申请上架</el-button
        >
        <el-button
          :disabled="slotProps.row.stockStatus !== '1'"
          type="text"
          @click="() => editBtn(slotProps.row)"
        >
          撤回申请
        </el-button>
        <!-- <span
            :class="
              slotProps.row.stockStatus == 0 ||
              slotProps.row.stockStatus == 1 ||
              slotProps.row.stockStatus == 2 ||
              slotProps.row.stockStatus == 3 ||
              slotProps.row.stockStatus == 5 ||
              slotProps.row.stockStatus == 6
                ? ''
                : 'btnColor'
            "
            >撤回申请</span
          > -->
        <el-button
          type="text"
          @click="() => del(slotProps.row)"
          :disabled="
            slotProps.row.stockStatus === '1' ||
              slotProps.row.stockStatus === '2'
          "
          >删除</el-button
        >
      </template>
    </tables>
    <div v-if="dialogFormVisible">
      <formDialog
        :rules="rules"
        :title="titles"
        :dialogFormVisible="dialogFormVisible"
        :searchData="authgroupForm"
        :searchForm="searchForm"
        @validateTrue="validateTrue"
        @validateFalse="validateFalse"
        @closeDialogFunc="closeDialog"
        ref="formDialog"
      >
      </formDialog>
    </div>
    <el-dialog
      title="单个导入"
      :visible.sync="centerDialogVisible"
      width="30%"
      center
      class="dialogStyle"
      :close-on-click-modal="false"
    >
      <el-input
        suffix-icon="el-icon-search"
        v-model="onceImportText"
        @keyup.enter.native="searchFundText"
        @input="inputFocus"
        size="small"
        clearable
      ></el-input>
      <ul
        v-if="showSearchList.length"
        class="ulSearchList"
        v-loading="!showSearchList.length"
      >
        <li
          v-for="(item, index) in showSearchList"
          :key="index"
          @click="SearchLiClick(item)"
        >
          {{ item.stkcode }} {{ item.stkname }}
        </li>
      </ul>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" size="mini" @click="uploadProStock"
          >确 定</el-button
        >
        <el-button @click="centerDialogVisible = false" size="mini"
          >取 消</el-button
        >
      </span>
    </el-dialog>
    <div v-if="centerDialogVisibleAll">
      <el-dialog
        title="批量导入"
        :visible.sync="centerDialogVisibleAll"
        width="30%"
        center
        class="dialogStyle"
        :close-on-click-modal="false"
      >
        <upload-cmp
          type="Stock"
          @getNewData="getNewData"
          :close.sync="centerDialogVisibleAll"
        />
      </el-dialog>
    </div>
    <div v-if="batchDialog">
      <el-dialog
        title="批量申请上架"
        :visible.sync="batchDialog"
        width="30%"
        center
        :close-on-click-modal="false"
      >
        <batchShelves
          type="Stock"
          :batchList="batchList"
          :close.sync="batchDialog"
        />
      </el-dialog>
    </div>
    <pagination
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
  </div>
</template>
<script>
import Pagination from '@/components/Pagination'
import uploadCmp from '@/components/upload'
import tables from '@/views/fundPool/components/table'
import formDialog from '@/views/fundPool/components/formDialog'
import batchShelves from '@/components/batchShelves'
import {
  getNotOnStockPage,
  batchPutOnApplyStock,
  batchWithdraPutOnApplyStock,
  batchDeleteStock,
  getListStock,
  addStockPool,
  importStock
} from '@/api/productStockPool/productStockPool'
import { getListMktSecondmarket } from '@/api/productMktFramework/index'
export default {
  name: 'stockPoolNotGrounding',
  components: {
    Pagination,
    tables,
    formDialog,
    uploadCmp,
    batchShelves
  },
  data () {
    return {
      batchList: '',
      batchDialog: false,
      groundingfundName: [],
      form: {
        tiems: '',
        stockStatus: ''
      },
      titlesearch: '',
      listQuery: {
        page: 1,
        limit: 10
      },
      total: 0,
      options: [
        {
          value: '0',
          label: '未上架'
        },
        {
          value: '1',
          label: '上架审核中'
        }
      ],
      tiemValue: [],
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
          prop: 'createDate',
          lable: '创建时间'
        },
        {
          prop: 'stockStatusCoyp',
          lable: '上下架状态'
        }
      ],
      optionsTable: [
        {
          name: '申请上架',
          action: row => this.watchBtn(row)
        },
        {
          name: '撤回申请',
          btnColor: '撤回申请',
          action: row => this.editBtn(row)
        },
        {
          name: '删除',
          action: row => this.del(row)
        }
      ],
      // form数据
      rules: {
        AssetSelect: {
          required: true,
          message: '请选择',
          trigger: 'blur'
        },
        description: {
          required: true,
          message: '请填写',
          trigger: 'blur'
        }
      },
      titles: '申请上架',
      dialogFormVisible: false,
      centerDialogVisibleAll: false,
      authgroupForm: {
        protocol: '',
        description: '',
        AssetSelect: ''
      },
      searchForm: [
        {
          type: 'Input',
          label: '上架产品',
          prop: 'protocol',
          width: '120px',
          readonly: true,
          isFormItemShow: true
        },
        {
          type: 'Select',
          label: '所属资产分类',
          prop: 'AssetSelect',
          width: '120px',
          props: [],
          isFormItemShow: true,
          change: row => {
            this.selectVal(row)
          }
        },
        {
          type: 'TextArea',
          label: '推荐理由',
          prop: 'description',
          width: '120px',
          isFormItemShow: true
        }
      ],
      roleList: [
        {
          value: '1',
          label: '医药'
        },
        {
          value: '2',
          label: '消费'
        },
        {
          value: '3',
          label: '科技'
        }
      ],
      centerDialogVisible: false,
      onceImportText: '',
      showSearchList: [],
      loading: true,
      checkList: [],
      AllfundCode: [],
      addStockPoolName: '',
      addStockPoolCode: '',
      exchangeMarket: '',
      tListMktSecondmarketArr: [],
      tListMktSecondmarketOpe: []
    }
  },
  watch: {
    centerDialogVisible (newVal, oldVal) {
      this.onceImportText = ''
      this.showSearchList = []
    },
    onceImportText (newVal, oldVal) {
      if (newVal === '') {
        this.showSearchList = []
      }
      if (newVal !== oldVal) {
        console.log(2)
      }
    },
    dialogFormVisible (newVal, oldVal) {
      if (!newVal) {
        this.checkList = []
        this.AllfundCode = []
        this.addStockPoolName = ''
        this.addStockPoolCode = ''
        this.tListMktSecondmarketOpe = []
        this.$refs.multipleTable.$children[0].clearSelection()
      }
    },
    tiemValue (newVal, old) {
      if (!newVal.length) {
        this.tiemValue = []
        this.getNotOnFndData()
      }
    }
  },
  created () {
    // this.searchForm.forEach(item => {
    //   if (item.props || item.props.length === 0) {
    //     item.props = this.roleList
    //   }
    // })
  },
  mounted () {
    this.getNotOnFndData()
  },
  methods: {
    getNewData () {
      this.getNotOnFndData()
      this.centerDialogVisibleAll = false
    },
    handleBatchShelves () {
      if (this.checkList.length > 0) {
        this.batchList = JSON.parse(JSON.stringify(this.checkList))
        this.batchDialog = true
      } else {
        this.dialogFormVisible = false
        this.$message.error('请勾选')
      }
    },
    getList () {
      this.getNotOnFndData()
    },
    // 请求列表数据
    getNotOnFndData () {
      getNotOnStockPage({
        key: this.titlesearch,
        page: this.listQuery.page,
        limit: this.listQuery.limit,
        offset: 0,
        stockStatus: this.form.stockStatus,
        startDate: this.tiemValue[0],
        endDate: this.tiemValue[1]
      })
        .then(res => {
          console.log(res)
          if (res.data.code === 0) {
            for (let i = 0; i < res.data.data.list.length; i++) {
              const item = res.data.data.list[i]
              const date = new Date(item.createDate)
              const y = date.getFullYear()
              let m = date.getMonth() + 1
              m = m < 10 ? '0' + m : m
              let d = date.getDate()
              d = d < 10 ? '0' + d : d
              const time = y + '-' + m + '-' + d
              item.createDate = time

              if (item.stockStatus == '0') {
                item.stockStatusCoyp = '未上架'
              }
              if (item.stockStatus == '1') {
                item.stockStatusCoyp = '上架审核中'
              }
            }
            this.tableData1 = res.data.data.list
            this.total = res.data.data.total
          }
        })
        .catch(error => {
          console.log(error)
        })
    },
    watchBtn (row) {
      console.log('申请上架')
      this.$refs.multipleTable.$children[0].clearSelection()
      if (
        row.stockStatus == '0' ||
        row.stockStatus == '5' ||
        row.stockStatus == '4'
      ) {
        this.dialogFormVisible = true
        this.checkList = [row]
        this.groundings()
      }
    },
    editBtn (row) {
      console.log(row)
      if (
        row.stockStatus == '0' ||
        row.stockStatus == '1' ||
        row.stockStatus == '2' ||
        row.stockStatus == '3' ||
        row.stockStatus == '5' ||
        '6'
      ) {
        this.$confirm('您是否确定撤销申请操作, 是否继续?', '', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            // this.dialogFormVisible = true
            this.checkList.push(row)
            this.batchWithdraPutOnApplyStockAll()
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消撤回申请'
            })
          })
      } else {
      }
    },
    del (row) {
      this.$refs.multipleTable.$children[0].clearSelection()
      this.checkList = [row]
      this.batchDeleteStockAll()
    },
    handleFilter () {
      this.getNotOnFndData()
    },
    // 申请上架
    validateTrue () {
      //  批量申请上架信息
      const AllfunCodeStr = this.AllfundCode.join(',')
      batchPutOnApplyStock({
        stkcode: AllfunCodeStr,
        assetsType: this.authgroupForm.AssetSelect,
        optMsg: this.authgroupForm.description
      }).then(res => {
        if (res.data.code == 0) {
          this.$message.success('申请成功!')
          this.getNotOnFndData()
          this.dialogFormVisible = false
        } else {
          this.$message.error(res.data.msg)
        }
        console.log(res)
      })
    },
    // 添加股票到股票池
    uploadProStock () {
      addStockPool({
        exchangeMarket: this.exchangeMarket,
        stkname: this.addStockPoolName,
        stkcode: this.addStockPoolCode
      })
        .then(res => {
          console.log(res)
          if (res.data.code === 0) {
            this.$message.success('添加成功!')
            this.getNotOnFndData()
          } else if (res.data.code === 10) {
            this.$message.error(res.data.msg)
          }
        })
        .catch(error => {
          console.log(error)
        })
      this.centerDialogVisible = false
    },
    validateFalse () {
      this.dialogFormVisible = true
    },
    closeDialog (newDialogFormVisible) {
      this.dialogFormVisible = newDialogFormVisible
    },
    selectVal (val) {
      console.log(val)
    },
    pickerChange () {
      this.getNotOnFndData()
    },
    fIconClick (scope) {
      console.log(scope)
      if (!scope.row.fwmsgShowstate === '0') {
        this.$confirm(
          `
        <div>当前触发预警为：${
          scope.row.fwmsgShowstate === '1' ? '黄色预警' : '红色预警'
        }</div><div>触发时间：${scope.row.createDate}</div>`,
          '预警提示',
          {
            dangerouslyUseHTMLString: true,
            confirmButtonText: '关闭',
            showCancelButton: false
            // center: true
          }
        )
      }
    },
    onceImport () {
      console.log(1)
      this.centerDialogVisible = true
      getListStock()
        .then(res => {
          this.showSearchList = res.data.data
        })
        .catch(error => {
          console.log(error)
        })
    },
    // 搜索股票信息
    searchFundText () {
      getListStock({
        key: String(this.onceImportText)
      })
        .then(res => {
          console.log(res)
          this.showSearchList = res.data.data
        })
        .catch(error => {
          console.log(error)
        })
      this.loading = false
    },
    SearchLiClick (item) {
      console.log(item)
      this.showSearchList = []
      this.onceImportText = item.stkname
      this.addStockPoolName = item.stkname
      this.addStockPoolCode = item.stkcode
      this.exchangeMarket = item.exchangeMarket
    },
    inputFocus () {
      this.loading = false
      this.searchFundText()
    },
    groundings () {
      this.dialogFormVisible = true
      this.searchForm = [
        {
          type: 'TextArea',
          label: '上架产品',
          prop: 'protocol',
          width: '120px',
          readonly: true,
          isFormItemShow: true
        },
        {
          type: 'Select',
          label: '所属资产分类',
          prop: 'AssetSelect',
          width: '120px',
          props: this.tListMktSecondmarketOpe,
          isFormItemShow: true,
          change: row => {
            this.selectVal2(row)
          }
        },
        {
          type: 'TextArea',
          label: '推荐理由',
          prop: 'description',
          width: '120px',
          isFormItemShow: true
        }
      ]
      this.titles = '申请上架'
      if (this.checkList.length) {
        this.groundingfundName = []
        this.AllfundCode = []
        this.checkList.forEach((item, index) => {
          this.groundingfundName.push(item.stkname)
          this.AllfundCode.push(item.stkcode)
        })
        this.authgroupForm.protocol = String(this.groundingfundName)
        // this.AllfundCode = this.checkList[0].fundCode
        console.log(this.checkList)
        this.getListMktSecondmarketFun()
      } else {
        this.dialogFormVisible = false
        this.$message.error('请勾选')
      }
    },
    getListMktSecondmarketFun () {
      getListMktSecondmarket().then(res => {
        console.log(res)
        this.tListMktSecondmarketArr = res.data.data
        this.tListMktSecondmarketArr.forEach(v => {
          this.tListMktSecondmarketOpe.push({
            value: v.tid,
            label: v.taaAssetName
          })
        })
        this.authgroupForm.AssetSelect = this.tListMktSecondmarketOpe[0].value
      })
    },
    groundingsAll () {
      this.centerDialogVisibleAll = true
    },
    // 撤回申请(批量、单个通用)
    batchWithdraPutOnApplyStockAll () {
      console.log(this.checkList)
      if (this.checkList.length) {
        this.checkList.forEach((item, index) => {
          this.groundingfundName.push(item.stkname)
          this.AllfundCode.push(item.stkcode)
        })
        this.authgroupForm.protocol = String(this.groundingfundName)
        // this.AllfundCode = this.checkList.stkcode
        const AllfunCodeStr = this.AllfundCode.join(',')
        batchWithdraPutOnApplyStock({
          stkcode: AllfunCodeStr
        }).then(res => {
          console.log(res)
          if (res.data.code == 0) {
            this.$message.success('撤回申请成功!')
            this.getNotOnFndData()
          } else {
            this.$refs.multipleTable.$children[0].clearSelection()
            this.$message.error('未上架!')
            this.checkList = []
            this.groundingfundName = []
            this.AllfundCode = []
          }
        })
      } else {
        this.$message.error('请勾选!')
      }
    },
    // 删除
    batchDeleteStockAll (row) {
      if (this.checkList.length) {
        this.$confirm('此操作将删除该产品, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.groundingfundName = []
          this.AllfundCode = []
          this.checkList.forEach((item, index) => {
            this.groundingfundName.push(item.stkname)
            this.AllfundCode.push(item.tid)
          })
          this.authgroupForm.protocol = String(this.groundingfundName)
          const AllfunCodeStr = this.AllfundCode.join(',')
          batchDeleteStock({
            tid: AllfunCodeStr
          }).then(res => {
            if (res.data.code == 0) {
              this.$message.success('删除成功!')
              this.getNotOnFndData()
            } else {
              this.$refs.multipleTable.$children[0].clearSelection()
              this.$message.error('删除失败!')
              this.checkList = []
              this.groundingfundName = []
              this.AllfundCode = []
            }
          })
        })
      } else {
        this.dialogFormVisible = false
        this.$message.error('请勾选!')
      }
    },
    selectVal2 (row) {
      console.log(row)
    },
    selectChange () {
      this.getNotOnFndData()
    },
    FgetCheckList (val) {
      this.checkList = val
      console.log(this.checkList)
    },
    // 文件上传
    onFileSuccess (data) {
      console.log(data)
      const {
        response: {
          data: { uploadId }
        }
      } = data
      // let fileName = data.response.data.fileAddr;

      importStock({ uploadId }).then(res => {
        console.log(res)
        if (res.data.code === 0) {
          this.centerDialogVisibleAll = false
          this.$message({
            type: 'success',
            message: res.data.msg
          })

          this.page = 1
          this.getDataTable()
        } else {
          this.$message.error(res.data.msg)
        }
      })
    },
    onClickUpload () {
      this.$refs.uploadCmp.uploadEvent()
    }
  }
}
</script>
<style lang="less">
.btnTop {
  margin: 50px 0;
}
.uploadBox {
  display: flex;
  .uploadText {
    width: 100%;
    line-height: 30px;
  }
  .uploadClik {
    margin-left: 20px;
    color: #1890ff;
    width: 120px;
    height: 30px;
  }
}
// .dialogStyle {
//   .el-dialog__body {
//     padding: 40px 100px;
//     margin-top: 20px;
//   }
//   .el-dialog--center .el-dialog__body {
//     text-align: center;
//   }
//   input {
//     border-radius: 10px;
//   }
//   .el-dialog--center .el-dialog__footer {
//     text-align: right;
//   }
// }
.ulSearchList {
  z-index: 99999;
  width: 100%;
  max-height: 272px;
  min-height: 76px;
  border: 1px solid #ccc;
  border-radius: 8px;
  background-color: #fff;
  color: #000;
  text-align: left;
  overflow: hidden;
  overflow: auto;
  li {
    padding: 5px 8px;
    font-size: 14px;
    color: #333;
    line-height: 20px;
  }
  li:hover {
    background-color: #e5f0fa;
    border-radius: 5px;
  }
}
.seachInput {
  display: flex;
  justify-content: space-between;
}
.btnColor {
  color: #000;
}
</style>
