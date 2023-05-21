<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="考试名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="考试名称"></el-input>
      </el-form-item>
<!--      <el-form-item label="试卷ID" prop="paperId">-->
<!--        <el-input v-model="dataForm.paperId" placeholder="试卷ID"></el-input>-->
<!--      </el-form-item>-->
      <el-form-item label="试卷" prop="paperId">
        <el-select v-model="dataForm.paperId" placeholder="请选择试卷">
          <el-option v-for="item in paperList" :value="item.id" :label="item.name" :key="item.id">
          </el-option>
        </el-select>
      </el-form-item>

<!--      <el-form-item label="合格分数" prop="qualifiedScore">-->
<!--        <el-input v-model="dataForm.qualifiedScore" placeholder="合格分数"></el-input>-->
<!--      </el-form-item>-->
<!--      <el-form-item label="是否删除  0 未删除 1已删除" prop="deleted">-->
<!--        <el-input v-model="dataForm.deleted" placeholder="是否删除  0 未删除 1已删除"></el-input>-->
<!--      </el-form-item>-->
<!--      <el-form-item label="创建时间" prop="createTime">-->
<!--        <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>-->
<!--      </el-form-item>-->
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
      paperList: [],
      visible: false,
      dataForm: {
        id: 0,
        name: '',
        paperId: '',
        qualifiedScore: '',
        deleted: '',
        createTime: ''
      },
      dataRule: {
        name: [
          {required: true, message: '考试名称不能为空', trigger: 'blur'}
        ],
        paperId: [
          {required: true, message: '试卷ID不能为空', trigger: 'blur'}
        ],
        qualifiedScore: [
          {required: true, message: '合格分数不能为空', trigger: 'blur'}
        ],
        deleted: [
          {required: true, message: '是否删除  0 未删除 1已删除不能为空', trigger: 'blur'}
        ],
        createTime: [
          {required: true, message: '创建时间不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  mounted () {
    this.getPapers()
  },
  methods: {
    getPapers () {
      this.$http({
        url: this.$http.adornUrl(`/exam/paper/listAll`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.paperList = data.data
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
            url: this.$http.adornUrl(`/exam/exam/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.exam.name
              this.dataForm.paperId = data.exam.paperId
              this.dataForm.qualifiedScore = data.exam.qualifiedScore
              this.dataForm.deleted = data.exam.deleted
              this.dataForm.createTime = data.exam.createTime
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
            url: this.$http.adornUrl(`/exam/exam/${!this.dataForm.id ? 'save' : 'update'}`),
            method: 'post',
            data: this.$http.adornData({
              'id': this.dataForm.id || undefined,
              'name': this.dataForm.name,
              'paperId': this.dataForm.paperId,
              'qualifiedScore': this.dataForm.qualifiedScore,
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
