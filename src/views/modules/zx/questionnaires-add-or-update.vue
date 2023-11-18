<template>
  <div>
    <el-dialog
      :title="!dataForm.id ? '新增' : '修改'"
      :close-on-click-modal="false"
      :visible.sync="visible">
      <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
      <el-form-item label="问卷名称" prop="questionnairesName">
        <el-input v-model="dataForm.questionnairesName" placeholder="问卷名称"></el-input>
      </el-form-item>
      <el-form-item label="客户类型" prop="customerType">
        <el-radio v-model="dataForm.customerType" :label="0">个人</el-radio>
        <el-radio v-model="dataForm.customerType" :label="1">公司</el-radio>
      </el-form-item>
      <el-form-item label="业务类型" prop="businessType">
        <el-radio v-model="dataForm.businessType" :label="0">理财</el-radio>
        <el-radio v-model="dataForm.businessType" :label="1">基金</el-radio>
        <el-radio v-model="dataForm.businessType" :label="2">资管</el-radio>
        <el-radio v-model="dataForm.businessType" :label="3">信托</el-radio>
      </el-form-item>
        <el-form-item label="所含题目" prop="businessType">
          <el-table ref="multipleTable" :data="questionnairesQuestionList" tooltip-effect="dark"
            style="width: 100%" @selection-change="handleSelectedChange">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column
              prop="questionContent"
              header-align="center"
              align="left"
              label="题目">
            </el-table-column>
            <el-table-column
              fixed="right"
              header-align="center"
              align="center"
              width="150"
              label="操作">
              <template slot-scope="scope">
<!--                <el-button type="text" size="small" @click="updateQuestionHandle(scope.row.id)">修改</el-button>-->
                <el-button type="text" size="small" @click="deleteQuestionHandle(scope.row.id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
          <div style="margin-top: 20px">
<!--            <el-button @click="deleteSelectedQuestions()">删除选中</el-button>-->
            <el-button @click="addQuestions()">添加题目</el-button>
          </div>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
      </span>
    </el-dialog>
    <el-dialog :visible="questionsSelectVisible"
               :visible.sync="questionsSelectVisible"
               :close-on-click-modal="false">
      <el-form :inline="true" @keyup.enter.native="getQuestionDataList()">
        <el-form-item>
          <el-input v-model="questionContentKey" placeholder="题目名称" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button @click="getQuestionDataList()">查找</el-button>
        </el-form-item>
      </el-form>
      <el-table :data="questionAllList" border v-loading="questionAllListLoading"
                @selection-change="questionSelectionChangeHandle" style="width: 100%;">
        <el-table-column
          type="selection"
          header-align="center"
          align="center"
          width="50">
        </el-table-column>

        <el-table-column
          prop="questionContent"
          header-align="center"
          align="center"
          label="题目">
        </el-table-column>
<!--        <el-table-column-->
<!--          fixed="right"-->
<!--          header-align="center"-->
<!--          align="center"-->
<!--          width="150"-->
<!--          label="操作">-->
<!--          <template slot-scope="scope">-->
<!--            <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>-->
<!--            <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>-->
<!--          </template>-->
<!--        </el-table-column>-->
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="questionsSelectVisible = false">取消</el-button>
        <el-button type="primary" @click="selectedQuestionHandle()">确定</el-button>
      </span>
      <el-pagination
        @size-change="questionSizeChangeHandle"
        @current-change="questionCurrentChangeHandle"
        :current-page="questionPageIndex"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="questionPageSize"
        :total="questionAllPage"
        layout="total, sizes, prev, pager, next, jumper">
      </el-pagination>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        // 问题选择器的数据
        questionPageIndex: 0,
        questionPageSize: 10,
        questionAllPage: 10,
        questionContentKey: '',
        questionsSelectVisible: false,
        questionAllListLoading: true,
        questionsSelectedList: [],
        questionAllList: [],

        // 问卷数据
        questionnairesQuestionList: [],
        questionnairesSelectedQuestionList: [], // 暂时不用

        dataForm: {
          id: 0,
          selectQuestions: '',
          questionnairesName: '',
          customerType: 0,
          businessType: 0,
          createTime: '',
          updateTime: '',
          isDeleted: '',
          questions: [
            {
              id: 0,
              questionContent: ''
            }
          ]
        },
        dataRule: {
          questionnairesName: [
            { required: true, message: '问卷名称不能为空', trigger: 'blur' }
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
                this.questionnairesQuestionList = data.questionnaires.questionList
              }
            })
          }
        })
      },
      // 获取题目数据
      getQuestionDataList () {
        this.questionAllListLoading = true
        this.$http({
          url: this.$http.adornUrl('/zx/questions/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.questionPageIndex,
            'limit': this.questionPageSize,
            'questionContentKey': this.questionContentKey
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.questionAllList = data.page.list
            this.questionAllPage = data.page.totalCount

            // 剔除问卷中已经选择的题目
            var selectList = this.questionnairesQuestionList
            this.questionAllList = this.questionAllList.filter(function (value, index, arr) {
              for (var i in selectList) {
                if (selectList[i].id === value.id) {
                  return false
                }
              }
              return true
            })
          } else {
            this.questionAllList = []
            this.questionAllPage = 0
          }
          this.questionAllListLoading = false
        })
      },
      // 每页数
      questionSizeChangeHandle (val) {
        this.questionPageSize = val
        this.questionPageIndex = 1
        this.getQuestionDataList()
      },
      // 当前页
      questionCurrentChangeHandle (val) {
        this.questionPageIndex = val
        this.getQuestionDataList()
      },
      questionSelectionChangeHandle (val) {
        this.questionsSelectedList = val
      },
      deleteSelectedQuestions () {
        this.questionnairesSelectedQuestionList = []
      },
      deleteQuestionHandle (id) {
        // 点击删除按钮删除
        this.questionnairesQuestionList = this.questionnairesQuestionList.filter(
          function (value, index, arr) {
            if (value.id === id) {
              return false
            }
            return true
          }
        )
      },
      addQuestions () {
        this.questionsSelectVisible = true
        this.getQuestionDataList()
      },
      selectedQuestionHandle () {
        for (var i in this.questionsSelectedList) {
          this.questionnairesQuestionList.push(this.questionsSelectedList[i])
        }
        this.questionsSelectVisible = false
      },
      handleSelectedChange (val) {
        this.questionnairesSelectedQuestionList = val
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
                'questionList': this.questionnairesQuestionList
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
