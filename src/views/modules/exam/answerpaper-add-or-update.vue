<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="学生id" prop="userId">
      <el-input v-model="dataForm.userId" placeholder="学生id"></el-input>
    </el-form-item>
    <el-form-item label="试卷id" prop="paperId">
      <el-input v-model="dataForm.paperId" placeholder="试卷id"></el-input>
    </el-form-item>
    <el-form-item label="题目id" prop="questionId">
      <el-input v-model="dataForm.questionId" placeholder="题目id"></el-input>
    </el-form-item>
    <el-form-item label="解析出来的答案" prop="answer">
      <el-input v-model="dataForm.answer" placeholder="解析出来的答案"></el-input>
    </el-form-item>
    <el-form-item label="学生的答案图片" prop="answerUrl">
      <el-input v-model="dataForm.answerUrl" placeholder="学生的答案图片"></el-input>
    </el-form-item>
    <el-form-item label="该题的答案得分" prop="score">
      <el-input v-model="dataForm.score" placeholder="该题的答案得分"></el-input>
    </el-form-item>
    <el-form-item label="是否删除  0 未删除 1已删除" prop="deleted">
      <el-input v-model="dataForm.deleted" placeholder="是否删除  0 未删除 1已删除"></el-input>
    </el-form-item>
    <el-form-item label="创建时间" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>
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
          userId: '',
          paperId: '',
          questionId: '',
          answer: '',
          answerUrl: '',
          score: '',
          deleted: '',
          createTime: ''
        },
        dataRule: {
          userId: [
            { required: true, message: '学生id不能为空', trigger: 'blur' }
          ],
          paperId: [
            { required: true, message: '试卷id不能为空', trigger: 'blur' }
          ],
          questionId: [
            { required: true, message: '题目id不能为空', trigger: 'blur' }
          ],
          answer: [
            { required: true, message: '解析出来的答案不能为空', trigger: 'blur' }
          ],
          answerUrl: [
            { required: true, message: '学生的答案图片不能为空', trigger: 'blur' }
          ],
          score: [
            { required: true, message: '该题的答案得分不能为空', trigger: 'blur' }
          ],
          deleted: [
            { required: true, message: '是否删除  0 未删除 1已删除不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建时间不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/exam/answerpaper/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userId = data.answerPaper.userId
                this.dataForm.paperId = data.answerPaper.paperId
                this.dataForm.questionId = data.answerPaper.questionId
                this.dataForm.answer = data.answerPaper.answer
                this.dataForm.answerUrl = data.answerPaper.answerUrl
                this.dataForm.score = data.answerPaper.score
                this.dataForm.deleted = data.answerPaper.deleted
                this.dataForm.createTime = data.answerPaper.createTime
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
              url: this.$http.adornUrl(`/exam/answerpaper/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userId': this.dataForm.userId,
                'paperId': this.dataForm.paperId,
                'questionId': this.dataForm.questionId,
                'answer': this.dataForm.answer,
                'answerUrl': this.dataForm.answerUrl,
                'score': this.dataForm.score,
                'deleted': this.dataForm.deleted,
                'createTime': this.dataForm.createTime
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
