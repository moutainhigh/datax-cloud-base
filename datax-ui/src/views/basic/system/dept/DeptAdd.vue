<template>
  <el-card class="box-card" shadow="always">
    <div slot="header" class="clearfix">
      <span>{{ title }}</span>
      <el-button-group style="float: right;">
        <el-button v-hasPerm="['system:dept:add']" size="mini" icon="el-icon-plus" round :loading="loadingOptions.loading" :disabled="loadingOptions.isDisabled" @click="submitForm">{{ loadingOptions.loadingText }}</el-button>
        <el-button size="mini" icon="el-icon-back" round @click="showCard">返回</el-button>
      </el-button-group>
    </div>
    <div class="body-wrapper">
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="上级部门" prop="parentId">
          <treeselect v-model="form.parentId" :options="deptOptions" :normalizer="normalizer" placeholder="请选择上级部门" />
        </el-form-item>
        <el-form-item label="部门名称" prop="deptName">
          <el-input v-model="form.deptName" placeholder="请输入部门名称" />
        </el-form-item>
        <el-form-item label="部门编码" prop="deptNo">
          <el-input v-model="form.deptNo" placeholder="请输入部门编码" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio
              v-for="dict in statusOptions"
              :key="dict.id"
              :label="dict.itemText"
            >{{ dict.itemValue }}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="form.remark" type="textarea" placeholder="请输入内容" />
        </el-form-item>
      </el-form>
    </div>
  </el-card>
</template>

<script>
import { addDept, listDept } from '@/api/system/dept'
import { construct } from '@/utils/json-tree'
import Treeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'

export default {
  name: 'DeptAdd',
  components: { Treeselect },
  props: {
    data: {
      type: Object,
      default: function() {
        return {}
      }
    }
  },
  data() {
    return {
      title: '部门新增',
      // 展示切换
      showOptions: {
        data: {},
        showList: true,
        showAdd: false,
        showEdit: false,
        showDetail: false
      },
      // 保存按钮
      loadingOptions: {
        loading: false,
        loadingText: '保存',
        isDisabled: false
      },
      // 表单参数
      form: {
        status: '1',
        parentId: '0'
      },
      // 表单校验
      rules: {
        parentId: [
          { required: true, message: '上级部门不能为空', trigger: 'blur' }
        ],
        deptName: [
          { required: true, message: '部门名称不能为空', trigger: 'blur' }
        ],
        deptNo: [
          { required: true, message: '部门编码不能为空', trigger: 'blur' }
        ]
      },
      // 状态数据字典
      statusOptions: [],
      // 部门树选项
      deptOptions: []
    }
  },
  created() {
    console.log('data:' + JSON.stringify(this.data))
    this.getDicts('sys_common_status').then(response => {
      if (response.success) {
        this.statusOptions = response.data
      }
    })
    this.getDeptTreeSelect()
  },
  methods: {
    showCard() {
      this.$emit('showCard', this.showOptions)
    },
    getDeptTreeSelect: function() {
      listDept().then(response => {
        if (response.success) {
          const { data } = response
          this.deptOptions = []
          const dept = { id: '0', deptName: '主类目', children: [] }
          dept.children = construct(data, {
            id: 'id',
            pid: 'parentId',
            children: 'children'
          })
          this.deptOptions.push(dept)
        }
      })
    },
    /** 转换部门数据结构 */
    normalizer(node) {
      if (node.children && !node.children.length) {
        delete node.children
      }
      return {
        id: node.id,
        label: node.deptName,
        children: node.children
      }
    },
    /** 提交按钮 */
    submitForm: function() {
      this.$refs['form'].validate(valid => {
        if (valid) {
          this.loadingOptions.loading = true
          this.loadingOptions.loadingText = '保存中...'
          this.loadingOptions.isDisabled = true
          addDept(this.form).then(response => {
            if (response.success) {
              this.$message.success('保存成功')
              setTimeout(() => {
                // 2秒后跳转列表页
                this.$emit('showCard', this.showOptions)
              }, 2000)
            } else {
              this.$message.error('保存失败')
              this.loadingOptions.loading = false
              this.loadingOptions.loadingText = '保存'
              this.loadingOptions.isDisabled = false
            }
          }).catch(() => {
            this.loadingOptions.loading = false
            this.loadingOptions.loadingText = '保存'
            this.loadingOptions.isDisabled = false
          })
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.el-card ::v-deep .el-card__body {
  height: calc(100vh - 230px);
  overflow-y: auto;
}
</style>
