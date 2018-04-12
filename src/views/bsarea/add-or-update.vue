<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="地区编码" prop="areaCd">
        <el-input v-model="dataForm.areaCd" placeholder="地区编码"></el-input>
      </el-form-item>
      <el-form-item label="名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="名称"></el-input>
      </el-form-item>
      <el-form-item label="上级ID" prop="parentId">
        <el-input v-model="dataForm.parentId" placeholder="上级ID"  ></el-input>
      </el-form-item>
      <el-form-item label="层级" prop="levels">
        <el-input v-model="dataForm.levels" placeholder="层级"></el-input>
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
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          areaCd: '',
          parentId: '',
          levels: '',
          name: ''
        },
        dataRule: {
          areaCd: [
            { required: true, message: '地区编码不能为空', trigger: 'blur' }
          ],
          name: [
            { required: true, message: '名称不能为空', trigger: 'blur' }
          ],
          parentId: [
            { required: true, message: '上级ID不能为空', trigger: 'blur' }
          ],
          levels: [
            { required: true, message: '层级不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id, parentId, levels) {
        this.dataForm.id = id || 0
        this.dataForm.parentId = parentId
        levels = parseInt(levels) + 1
        if (levels === 4) {
          this.dataForm.leaf = 1
        } else {
          this.dataForm.leaf = 0
        }
        this.dataForm.levels = levels
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            API.bsarea.info(this.dataForm.id).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.areaCd = data.bsArea.areaCd
                this.dataForm.parentId = data.bsArea.parentId
                this.dataForm.levels = data.bsArea.levels
                this.dataForm.name = data.bsArea.name
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            var params = {
              'id': this.dataForm.id || undefined,
              'areaCd': this.dataForm.areaCd,
              'name': this.dataForm.name,
              'parentId': this.dataForm.parentId,
              'levels': this.dataForm.levels,
              'leaf': this.dataForm.leaf
            }
            var tick = !this.dataForm.id ? API.bsarea.add(params) : API.bsarea.update(params)
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
