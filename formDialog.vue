<template>
  <div class="ces-search">
    <el-dialog
      center
      :title="title"
      :visible.sync="dialogFormVisible"
      :before-close="handleFormModalCancel"
      :width="dialogwidth"
      customClass="modalClass roleClass"
    >
      <!-- <div slot="title" class="dialog-footer" v-if="title">
        <slot name="title">{{ title }}</slot>
      </div> -->
      <el-form
        :size="size"
        inline
        :label-width="labelWidth"
        :model="searchData"
        ref="searchData"
        :rules="rules"
        class="formClass"
      >
        <template v-for="item in searchForm">
          <el-form-item
            v-if="item.isFormItemShow"
            :label="item.label"
            :key="item.prop"
            :prop="item.prop"
            :label-width="item.width"
          >
            <!-- input -->
            <el-input
              v-if="item.type === 'Input'"
              v-model="searchData[item.prop]"
              size="mini"
              :placeholder="item.placeholder"
              :readonly="item.readonly"
              @focus="item.focus()"
              style="width:190px"
            ></el-input>
            <!-- select -->
            <el-select
              v-if="item.type === 'Select'"
              v-model="searchData[item.prop]"
              :placeholder="item.placeholder"
              size="mini"
              @change="item.change(searchData[item.prop])"
              style="width:190px"
            >
              <el-option
                v-for="op in item.props"
                :label="op.label"
                :value="op.value"
                :key="op.value"
              ></el-option>
            </el-select>
            <!-- textarea-->
            <el-input
              v-if="item.type === 'TextArea'"
              v-model="searchData[item.prop]"
              type="textarea"
              maxlength="100"
              minlength="1"
              size="small"
              :placeholder="item.placeholder"
              :disabled="item.disabled"
              :readonly="item.readonly"
              show-word-limit
              style="width:190px"
              autosize
            ></el-input>
            <!-- <div v-if="item.type === 'tag'">
              {{ searchData[item.prop] }}
            </div> -->
            <!--tree-->
            <el-popover
              placement="bottom"
              trigger="click"
              v-model="popoverDialog"
              v-if="item.type === 'tree2'"
            >
              <choiceIndex
                :treesData="item.props"
                :showLabel="false"
                @choicedIndex="choicedIndex"
              />
              <el-input
                slot="reference"
                size="small"
                :value="searchData.monindicIdname"
                placeholder=""
              ></el-input>
              <!-- <el-tree
                    :data="item.props"
                    :props="defaultProps"
                    :filter-node-method="filterNode"
                    @node-click="handleNodeClick"
                  ></el-tree> -->
            </el-popover>
            <el-tree
              v-if="item.type === 'Tree'"
              ref="tree"
              :data="item.props"
              show-checkbox
              default-expand-all
              node-key="id"
              highlight-current
              :props="defaultProps"
              :default-checked-keys="checkedDedault"
              @check-change="getCheckedKeys"
            />
          </el-form-item>
        </template>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="handleFormModalCancel('searchData')" size="mini"
          >取 消</el-button
        >
        <el-button
          type="primary"
          @click="handleFormModalOk('searchData')"
          size="mini"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
import choiceIndex from '@/views/strategy/components/createPlan/choiceIndex.vue'
export default {
  components: {
    choiceIndex
  },
  props: {
    rules: {
      type: Object
    },
    title: {
      type: String
    },
    dialogwidth: {
      type: String,
      default: '420px'
    },
    labelWidth: {
      type: String,
      default: '80px'
    },
    size: {
      type: String,
      default: 'mini'
    },
    searchForm: {
      type: Array
    },
    searchData: {
      type: Object
    },

    dialogFormVisible: {
      type: Boolean,
      default: false
    },
    checkedDedault: {
      type: Array
    },
    openType: {
      // default: '',
      type: String
    }
  },
  data () {
    return {
      newDialogFormVisible: false,
      newFormEditType: '',
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      popoverDialog: false
      // tree2Data: ''
    }
  },
  watch: {
    dialogFormVisible: {
      handler (newVal, oldVal) {
        this.newDialogFormVisible = newVal
        if (this.$refs.searchData && newVal === true) {
          this.$refs.searchData.resetFields()
        } else {
          console.log(1)
        }
      }
    }
  },

  methods: {
    handleFormModalOk (searchData) {
      this.$refs.searchData.validate(valid => {
        if (valid) {
          this.$emit('validateTrue', searchData)
          // this.$refs.searchData.resetFields()
        } else {
          this.$emit('validateFalse', searchData)
        }
      })
    },
    handleFormModalCancel (searchData) {
      this.newDialogFormVisible = false
      this.$refs.searchData.resetFields()
      this.$emit('closeDialogFunc', this.newDialogFormVisible)
    },
    getCheckedKeys (data, checked, indeterminate) {
      var roleTemp = []
      var roleArr = this.$refs.tree[0]
        .getHalfCheckedKeys()
        .concat(this.$refs.tree[0].getCheckedKeys())

      for (var i = 0; i < roleArr.length; i++) {
        roleTemp.push({ id: roleArr[i] })
      }
      this.$emit('getCheckedKeys', roleTemp)
    },
    // 查找树节点
    filterNode (value, data) {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    },
    // 点击树节点
    handleNodeClick (obj, nodes) {
      if (nodes.childNodes.length === 0) {
        this.$emit('choicedIndex', obj)
      }
    },
    choicedIndex (obj) {
      // console.log(arguments)
      // this.tree2Data = obj.itemObj.screenName
      this.$emit('selectMonindicId', obj.itemObj)
      this.popoverDialog = false
    }
  }
  // created () {
  //   // console.log(this)
  //   // if (this.openType === 'edit') {
  //   //   }
  //   // this.tree2Data = this.searchData.screenName
  //   // console.log(this.tree2Data)
  // }
}
</script>
<style lang="less">
.el-dialog {
  border-radius: 8px;
}
</style>
