<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
<!--    <el-form-item label="课程id" prop="courseId">-->
<!--      <el-input v-model="dataForm.courseId" placeholder="课程id"></el-input>-->
<!--    </el-form-item>-->
      <el-form-item label="所属课程" prop="courseId">
<!--      <el-input v-model="dataForm.courseId" placeholder="课程id"></el-input>-->
        <el-select v-model="dataForm.courseId" placeholder="请选择所属课程">
          <el-option v-for="item in courseList" :value="item.id" :label="item.name" :key="item.id">
          </el-option>
        </el-select>
    </el-form-item>
    <el-form-item label="题目类型" prop="type">
      <el-input v-model="dataForm.type" disabled placeholder="题目类型"></el-input>
    </el-form-item>
<!--    <el-form-item label="题目图片" prop="imgurl">-->
<!--      <el-input v-model="dataForm.imgurl" placeholder="题目图片"></el-input>-->
<!--    </el-form-item>-->
    <el-form-item label="题干" prop="content">
      <el-input v-model="dataForm.content" placeholder="请输入题干"></el-input>
    </el-form-item>
    <el-form-item label="答案" prop="answer">
      <el-input v-model="dataForm.answer" placeholder="标准答案"></el-input>
    </el-form-item>
    <el-form-item label="题目分数" prop="scores">
      <el-input type="number" v-model="dataForm.scores" placeholder="题目分数"></el-input>
    </el-form-item>
<!--    <el-form-item label="是否删除  0 未删除 1已删除" prop="deleted">-->
<!--      <el-input v-model="dataForm.deleted" placeholder="是否删除  0 未删除 1已删除"></el-input>-->
<!--    </el-form-item>-->
<!--    <el-form-item label="创建时间" prop="createTime">-->
<!--      <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>-->
<!--    </el-form-item>-->
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
        courseList: [],
        visible: false,
        dataForm: {
          id: 0,
          courseId: '',
          type: '简答题',
          imgurl: '',
          content: '',
          answer: '',
          scores: '',
          deleted: '',
          createTime: ''
        },
        dataRule: {
          courseId: [
            { required: true, message: '所属课程不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '题目类型不能为空', trigger: 'blur' }
          ],
          imgurl: [
            { required: true, message: '题目图片不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '题目不能为空', trigger: 'blur' }
          ],
          answer: [
            { required: true, message: '答案不能为空', trigger: 'blur' }
          ],
          scores: [
            { required: true, message: '题目满分分数不能为空', trigger: 'blur' }
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
    mounted () {
      this.getCourse()
    },
    methods: {
      getCourse () {
        this.$http({
          url: this.$http.adornUrl(`/generator/ecourse/list`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.courseList = data.page.list
          }
        })
      },
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/exam/question/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.courseId = data.question.courseId
                this.dataForm.type = data.question.type
                this.dataForm.imgurl = data.question.imgurl
                this.dataForm.content = data.question.content
                this.dataForm.answer = data.question.answer
                this.dataForm.scores = data.question.scores
                this.dataForm.deleted = data.question.deleted
                this.dataForm.createTime = data.question.createTime
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
              url: this.$http.adornUrl(`/exam/question/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'courseId': this.dataForm.courseId,
                'type': this.dataForm.type,
                'imgurl': this.dataForm.imgurl,
                'content': this.dataForm.content,
                'answer': this.dataForm.answer,
                'scores': this.dataForm.scores,
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
