<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="试卷名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="试卷名称"></el-input>
    </el-form-item>
    <el-form-item label="试卷总数" prop="totalScore">
      <el-input v-model="dataForm.totalScore" placeholder="试卷总数"></el-input>
    </el-form-item>
    <el-form-item label="试卷总时长" prop="totalTime">
      <el-input v-model="dataForm.totalTime" placeholder="试卷总时长"></el-input>
    </el-form-item>
    <el-form-item label="学院id" prop="majorId">
      <el-input v-model="dataForm.majorId" placeholder="学院id"></el-input>
    </el-form-item>
    <el-form-item label="课程id" prop="courseId">
      <el-input v-model="dataForm.courseId" placeholder="课程id"></el-input>
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
          name: '',
          totalScore: '',
          totalTime: '',
          majorId: '',
          courseId: '',
          deleted: '',
          createTime: ''
        },
        dataRule: {
          name: [
            { required: true, message: '试卷名称不能为空', trigger: 'blur' }
          ],
          totalScore: [
            { required: true, message: '试卷总数不能为空', trigger: 'blur' }
          ],
          totalTime: [
            { required: true, message: '试卷总时长不能为空', trigger: 'blur' }
          ],
          majorId: [
            { required: true, message: '学院id不能为空', trigger: 'blur' }
          ],
          courseId: [
            { required: true, message: '课程id不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/exam/paper/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.paper.name
                this.dataForm.totalScore = data.paper.totalScore
                this.dataForm.totalTime = data.paper.totalTime
                this.dataForm.majorId = data.paper.majorId
                this.dataForm.courseId = data.paper.courseId
                this.dataForm.deleted = data.paper.deleted
                this.dataForm.createTime = data.paper.createTime
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
              url: this.$http.adornUrl(`/exam/paper/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'totalScore': this.dataForm.totalScore,
                'totalTime': this.dataForm.totalTime,
                'majorId': this.dataForm.majorId,
                'courseId': this.dataForm.courseId,
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
