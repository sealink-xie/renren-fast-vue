<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">

      <el-form-item label="部门名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="部门名称"></el-input>
      </el-form-item>
      <el-form-item label="上级部门" prop="parentName">
        <el-popover
          ref="menuListPopover"
          placement="bottom-end"
          trigger="click">
          <el-tree
            :data="menuList"
            :props="menuListTreeProps"
            node-key="menuId"
            ref="menuListTree"
            @current-change="menuListTreeCurrentChangeHandle"
            :default-expand-all="true"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.parentName" v-popover:menuListPopover :readonly="true" placeholder="点击选择上级菜单" class="menu-list__input"></el-input>
      </el-form-item>

      <el-form-item v-if="dataForm.type !== 2" label="排序号" prop="orderNum">
        <el-input-number v-model="dataForm.orderNum" controls-position="right" :min="0" label="排序号"></el-input-number>
      </el-form-item>
    </el-form>

    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import API from '@/api'
  import { treeDataTranslate } from '@/utils'

  export default {
    data () {
//      var validateUrl = (rule, value, callback) => {
//        if (this.dataForm.type === 1 && !/\S/.test(value)) {
//          callback(new Error('菜单URL不能为空'))
//        } else {
//          callback()
//        }
//      }
      return {
        visible: false,
        dataForm: {
          id: 0,
          type: 1,
          name: '',
          parentId: 0,
          parentName: '',
          url: '',
          perms: '',
          orderNum: 0,
          icon: ''
        },
        dataRule: {
          name: [
            { required: true, message: '部门名称不能为空', trigger: 'blur' }
          ],
  //        url: [
  //          { validator: validateUrl, trigger: 'blur' }
 //         ],
          parentName: [
            { required: true, message: '上级部门不能为空', trigger: 'change' }
          ]
        },
        deptList: [],
        deptListTreeProps: {
          label: 'name',
          children: 'children'
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        API.dept.select().then(({data}) => {
          this.deptList = treeDataTranslate(data.deptList, 'deptId')
          alert('11')
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }).then(() => {
          if (!this.dataForm.id) {
            // 新增
            this.deptListTreeSetCurrentNode()
          } else {
            // 修改
            API.dept.info(this.dataForm.id).then(({data}) => {
              this.dataForm.id = data.dept.deptId
              this.dataForm.name = data.dept.name
              this.dataForm.parentId = data.dept.parentId
              this.dataForm.orderNum = data.dept.orderNum
              this.deptListTreeSetCurrentNode()
            })
          }
        })
      },
      // 菜单树选中
      deptListTreeCurrentChangeHandle (data, node) {
        this.dataForm.parentId = data.deptId
        this.dataForm.parentName = data.name
      },
      // 菜单树设置当前选中节点
      deptListTreeSetCurrentNode () {
        this.$refs.menuListTree.setCurrentKey(this.dataForm.parentId)
        this.dataForm.parentName = (this.$refs.menuListTree.getCurrentNode() || {})['name']
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            var params = {
              'deptId': this.dataForm.id || undefined,
              'name': this.dataForm.name,
              'parentId': this.dataForm.parentId,
              'orderNum': this.dataForm.orderNum
            }
            var tick = !this.dataForm.id ? API.dept.add(params) : API.dept.update(params)
            tick.then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>

<style lang="scss">
  .mod-menu {
    .menu-list__input {
       > .el-input__inner {
        cursor: pointer;
      }
    }
    .icon-tips {
      font-size: 18px;
      text-align: center;
      color: #e6a23c;
      cursor: pointer;
    }
  }
</style>
