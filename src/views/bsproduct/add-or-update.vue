<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="100px">
      <el-tabs  >
        <el-tab-pane label="基础信息"  >
          <el-form-item label="物料编码" prop="productCd">
            <el-input v-model="dataForm.productCd" placeholder="物料编码"></el-input>
          </el-form-item>
          <el-form-item label="品牌" prop="brand">
            <el-input v-model="dataForm.brand" placeholder="品牌"></el-input>
          </el-form-item>
          <el-form-item label="型号" prop="style">
            <el-input v-model="dataForm.style" placeholder="型号"></el-input>
          </el-form-item>
          <el-form-item label="色号" prop="colorCd">
            <el-input v-model="dataForm.colorCd" placeholder="色号"></el-input>
          </el-form-item>
          <el-form-item label="年份" prop="year">
            <el-input v-model="dataForm.year" placeholder="年份"></el-input>
          </el-form-item>
          <el-form-item label="总部库存" prop="stock">
            <el-input v-model="dataForm.stock" placeholder="总部库存"></el-input>
          </el-form-item>
          <el-form-item label="代理商库存" prop="agentStock">
            <el-input v-model="dataForm.agentStock" placeholder="代理商库存"></el-input>
          </el-form-item>
          <el-form-item label="安全库存" prop="safetyStock">
            <el-input v-model="dataForm.safetyStock" placeholder="安全库存"></el-input>
          </el-form-item>
        </el-tab-pane>

        <el-tab-pane label="参数信息">
          <el-form-item label="是否定制" prop="isCm">
            <el-input v-model="dataForm.isCm" placeholder="是否定制"></el-input>
          </el-form-item>
          <el-form-item label="备注" prop="remark">
            <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
          </el-form-item>
          <el-form-item label="参数1" prop="parameter1">
            <el-input v-model="dataForm.parameter1" placeholder="参数1"></el-input>
          </el-form-item>
          <el-form-item label="参数2" prop="parameter2">
            <el-input v-model="dataForm.parameter2" placeholder="参数2"></el-input>
          </el-form-item>
          <el-form-item label="参数3" prop="parameter3">
            <el-input v-model="dataForm.parameter3" placeholder="参数3"></el-input>
          </el-form-item>
          <el-form-item label="参数4" prop="parameter4">
            <el-input v-model="dataForm.parameter4" placeholder="参数4"></el-input>
          </el-form-item>
        </el-tab-pane>
      </el-tabs>
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
        tabPosition: 'right',
        dataForm: {
          id: 0,
          productCd: '',
          brand: '',
          style: '',
          colorCd: '',
          year: '',
          stock: '',
          agentStock: '',
          safetyStock: '',
          isCm: '',
          remark: '',
          parameter1: '',
          parameter2: '',
          parameter3: '',
          parameter4: '',
          parameter5: '',
          parameter6: ''
        },
        dataRule: {
          productCd: [
            { required: true, message: '物料编码不能为空', trigger: 'blur' }
          ],
          brand: [
            { required: true, message: '品牌不能为空', trigger: 'blur' }
          ],
          style: [
            { required: true, message: '型号不能为空', trigger: 'blur' }
          ],
          colorCd: [
            { required: true, message: '色号不能为空', trigger: 'blur' }
          ],
          year: [
            { required: true, message: '年份不能为空', trigger: 'blur' }
          ],
          stock: [
            { required: true, message: '总部库存不能为空', trigger: 'blur' }
          ],
          agentStock: [
            { required: true, message: '代理商库存不能为空', trigger: 'blur' }
          ],
          safetyStock: [
            { required: true, message: '安全库存不能为空', trigger: 'blur' }
          ],
          isCm: [
            { required: true, message: '是否定制不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            API.bsproduct.info(this.dataForm.id).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.productCd = data.bsProduct.productCd
                this.dataForm.brand = data.bsProduct.brand
                this.dataForm.style = data.bsProduct.style
                this.dataForm.colorCd = data.bsProduct.colorCd
                this.dataForm.year = data.bsProduct.year
                this.dataForm.stock = data.bsProduct.stock
                this.dataForm.agentStock = data.bsProduct.agentStock
                this.dataForm.safetyStock = data.bsProduct.safetyStock
                this.dataForm.isCm = data.bsProduct.isCm
                this.dataForm.remark = data.bsProduct.remark
                this.dataForm.parameter1 = data.bsProduct.parameter1
                this.dataForm.parameter2 = data.bsProduct.parameter2
                this.dataForm.parameter3 = data.bsProduct.parameter3
                this.dataForm.parameter4 = data.bsProduct.parameter4
                this.dataForm.parameter5 = data.bsProduct.parameter5
                this.dataForm.parameter6 = data.bsProduct.parameter6
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
              'productCd': this.dataForm.productCd,
              'brand': this.dataForm.brand,
              'style': this.dataForm.style,
              'colorCd': this.dataForm.colorCd,
              'year': this.dataForm.year,
              'stock': this.dataForm.stock,
              'agentStock': this.dataForm.agentStock,
              'safetyStock': this.dataForm.safetyStock,
              'isCm': this.dataForm.isCm,
              'remark': this.dataForm.remark,
              'parameter1': this.dataForm.parameter1,
              'parameter2': this.dataForm.parameter2,
              'parameter3': this.dataForm.parameter3,
              'parameter4': this.dataForm.parameter4,
              'parameter5': this.dataForm.parameter5,
              'parameter6': this.dataForm.parameter6
            }
            var tick = !this.dataForm.id ? API.bsproduct.add(params) : API.bsproduct.update(params)
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
