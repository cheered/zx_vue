<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="问卷名称" prop="questionnairesName">
      <el-input v-model="dataForm.questionnairesName" placeholder="问卷名称"></el-input>
    </el-form-item>
    <el-form-item label="客户类型" prop="customerType">
      <el-radio v-model="dataForm.customerType" label="1">个人</el-radio>
      <el-radio v-model="dataForm.customerType" label="2">公司</el-radio>
    </el-form-item>
    <el-form-item label="业务类型" prop="businessType">
      <el-radio v-model="dataForm.businessType" label="1">理财</el-radio>
      <el-radio v-model="dataForm.businessType" label="2">基金</el-radio>
      <el-radio v-model="dataForm.businessType" label="3">资管</el-radio>
      <el-radio v-model="dataForm.businessType" label="4">信托</el-radio>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          questionnairesName: '',
          customerType: '',
          businessType: '',
          createTime: '',
          updateTime: '',
          isDeleted: ''
        },
        dataRule: {
          questionnairesName: [
            { required: true, message: '问卷名称不能为空', trigger: 'blur' }
          ],
          customerType: [
            { required: true, message: '问卷面向的客户类型不能为空', trigger: 'blur' }
          ],
          businessType: [
            { required: true, message: '问卷面向的业务类型不能为空', trigger: 'blur' }
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
            this.$http({
              url: this.$http.adornUrl(`/zx/questionnaires/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.questionnairesName = data.questionnaires.questionnairesName
                this.dataForm.customerType = data.questionnaires.customerType
                this.dataForm.businessType = data.questionnaires.businessType
                this.dataForm.createTime = data.questionnaires.createTime
                this.dataForm.updateTime = data.questionnaires.updateTime
                this.dataForm.isDeleted = data.questionnaires.isDeleted
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/zx/questionnaires/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'questionnairesName': this.dataForm.questionnairesName,
                'customerType': this.dataForm.customerType,
                'businessType': this.dataForm.businessType,
                'createTime': this.dataForm.createTime,
                'updateTime': this.dataForm.updateTime,
                'isDeleted': this.dataForm.isDeleted
              })
            }).then(({data}) => {
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
