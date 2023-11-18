<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="题目ID" prop="questionId">
      <el-input v-model="dataForm.questionId" placeholder="题目ID"></el-input>
    </el-form-item>
    <el-form-item label="选项具体内容" prop="optionContent">
      <el-input v-model="dataForm.optionContent" placeholder="选项具体内容"></el-input>
    </el-form-item>
    <el-form-item label="选项得分" prop="optionScore">
      <el-input v-model="dataForm.optionScore" placeholder="选项得分"></el-input>
    </el-form-item>
    <el-form-item label="创建时间" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>
    </el-form-item>
    <el-form-item label="更新时间" prop="updateTime">
      <el-input v-model="dataForm.updateTime" placeholder="更新时间"></el-input>
    </el-form-item>
    <el-form-item label="逻辑删除标志" prop="isDeleted">
      <el-input v-model="dataForm.isDeleted" placeholder="逻辑删除标志"></el-input>
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
          questionId: '',
          optionContent: '',
          optionScore: '',
          createTime: '',
          updateTime: '',
          isDeleted: ''
        },
        dataRule: {
          questionId: [
            { required: true, message: '题目ID不能为空', trigger: 'blur' }
          ],
          optionContent: [
            { required: true, message: '选项具体内容不能为空', trigger: 'blur' }
          ],
          optionScore: [
            { required: true, message: '选项得分不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建时间不能为空', trigger: 'blur' }
          ],
          updateTime: [
            { required: true, message: '更新时间不能为空', trigger: 'blur' }
          ],
          isDeleted: [
            { required: true, message: '逻辑删除标志不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/zx/options/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.questionId = data.options.questionId
                this.dataForm.optionContent = data.options.optionContent
                this.dataForm.optionScore = data.options.optionScore
                this.dataForm.createTime = data.options.createTime
                this.dataForm.updateTime = data.options.updateTime
                this.dataForm.isDeleted = data.options.isDeleted
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
              url: this.$http.adornUrl(`/zx/options/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'questionId': this.dataForm.questionId,
                'optionContent': this.dataForm.optionContent,
                'optionScore': this.dataForm.optionScore,
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
