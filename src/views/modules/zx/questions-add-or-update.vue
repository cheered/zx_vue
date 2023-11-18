<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="题目" prop="questionContent">
      <el-input v-model="dataForm.questionContent" placeholder="题目"></el-input>
    </el-form-item>
    <el-form-item label="选项" prop="questionContent">
      <el-row style="margin-top: 5px" v-for="o in dataForm.options">
        <el-col :span="19">
          <el-input v-model="o.optionContent" placeholder="选项"></el-input>
        </el-col>
        <el-col :span="5">
          <el-input v-model="o.optionScore" placeholder="分值" type="number" min="0"></el-input>
        </el-col>
      </el-row>
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
          questionContent: '',
          createTime: '',
          updateTime: '',
          isDeleted: '',
          options: [
            {
              optionContent: '',
              optionScore: 0
            },
            {
              optionContent: '',
              optionScore: 0
            },
            {
              optionContent: '',
              optionScore: 0
            },
            {
              optionContent: '',
              optionScore: 0
            }
          ]
        },
        dataRule: {
          questionContent: [
            { required: true, message: '题目内容不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/zx/questions/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.questionContent = data.questions.questionContent
                this.dataForm.createTime = data.questions.createTime
                this.dataForm.updateTime = data.questions.updateTime
                this.dataForm.isDeleted = data.questions.isDeleted
                this.dataForm.options = data.questions.options
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
              url: this.$http.adornUrl(`/zx/questions/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'questionContent': this.dataForm.questionContent,
                'options': this.dataForm.options
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
