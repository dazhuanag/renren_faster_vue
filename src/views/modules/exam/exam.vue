<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="请输入考试名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('exam:exam:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('exam:exam:delete')" type="danger" @click="deleteHandle()"
                   :disabled="dataListSelections.length <= 0">批量删除
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="考试名称">
      </el-table-column>
      <el-table-column
        prop="paperName"
        header-align="center"
        align="center"
        label="试卷名称">
      </el-table-column>
      <el-table-column
        prop="totalScore"
        header-align="center"
        align="center"
        label="总分">
      </el-table-column>
      <el-table-column
        prop="totalTime"
        header-align="center"
        align="center"
        label="考试时长">
      </el-table-column>
      <el-table-column
        prop="deleted"
        header-align="center"
        align="center"
        label="是否删除">
        <template slot-scope="scope">
          <el-tag type="success" v-if="scope.row.deleted===0">未删除</el-tag>
          <el-tag type="danger" v-else>已删除</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('exam:answerpaper:save')" type="text" size="small" @click="joinExam(scope.row)">参加考试
          </el-button>
          <el-button v-if="isAuth('exam:exam:save')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">
            修改
          </el-button>
          <el-button v-if="isAuth('exam:exam:delete')" type="text" size="small" @click="deleteHandle(scope.row.id)">删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>

    <el-dialog title="参加考试" :visible.sync="dialogFormVisible">
      <el-form :model="examination" ref="examination">
        <el-form-item style="display:none;" label="testpaper" label-width="100px" prop="testpaper">
          <el-input v-model="examination.testpaper"></el-input>
        </el-form-item>
        <el-form-item label="考试名称：" label-width="100px">
          {{ examination.name }}
        </el-form-item>
        <el-form-item label="试卷名称：" label-width="100px">
          {{ examination.paperName }}
        </el-form-item>
        <el-form-item label="考试总分：" label-width="100px">
          {{ examination.totalScore }}
        </el-form-item>
        <el-form-item label="考试时间：" label-width="100px">
          {{ examination.totalTime }} 分钟
        </el-form-item>
        <el-form-item v-if="examination.type===1" label="输入口令：" label-width="100px">
          <el-input v-model="examination.password" clearable></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="surejoin()">确 定</el-button>
      </div>
    </el-dialog>

    <div class="testPaper">
      <el-dialog :visible.sync="dialogFormVisibleExam" :fullscreen="true">
        <!--      <div id="timerClass">-->
        <!--        距离考试结束还有：-->
        <!--        <div style="color:red;">{{ djs }}</div>-->
        <!--      </div>-->
        <div class="questionClass">
          <div class="testPaperHeader">
            <div style="font-size:20px;">{{ testPaper.name }}
              <el-button type="primary" @click="suresubmit()" style="float:right;margin-top: 10px;margin-left:20px;">交卷</el-button>
              <el-button type="primary" @click="dialogFormVisibleExam=false" style="float:right;margin-top: 10px;margin-left:20px;">退出
              </el-button>
            </div>
            <div>考试时间：{{ testPaper.totalTime }}分钟
              &nbsp;&nbsp;&nbsp;总分：{{ testPaper.totalScore }} 分
            </div>
          </div>
          <div v-for="(question,index) in testPaper.questionList" class="questionClass2">
            {{ index + 1 }}.{{ question.content }}   ( {{ question.score }} 分)
            <div style="margin-top:10px;margin-bottom: 20px;">
              <div>
                <el-input v-model="question.reply" type="textarea" :rows="5">
                </el-input>
<!--                <el-upload-->
<!--                  ref="uploadPic"-->
<!--                  action=""-->
<!--                  :http-request="Upload"-->
<!--                  :before-upload="BeforeUpload"-->
<!--                  :on-preview="handlePreview"-->
<!--                  :on-remove="handleRemove"-->
<!--                  :on-success="handleSuccess"-->
<!--                  :file-list="fileList"-->
<!--                  :limit = '1'-->
<!--                  :on-exceed = "handleExceed"-->
<!--                  :data = uploadData-->
<!--                  :auto-upload="true">-->
<!--                  <el-button slot="trigger" size="small" type="primary">选取文件</el-button>-->
<!--&lt;!&ndash;                  <el-button style="margin-left: 10px;" size="small" type="success" @click="Upload(question)">上传图片到服务器</el-button>&ndash;&gt;-->
<!--                </el-upload>-->
              </div>`
            </div>
          </div>
        </div>
      </el-dialog>
    </div>

  </div>
</template>

<script>
import AddOrUpdate from './exam-add-or-update'

export default {
  data () {
    return {
      dataForm: {
        key: ''
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      dialogFormVisible: false,
      dialogFormVisibleExam: false,
      examination: {
        id: '',
        name: '',
        paperName: '',
        totalTime: '',
        totalScore: '',
        paperId: ''
      },
      questions: [],
      testPaper: {},
      answer: {
        courseName: '',
        courseType: '',
        courseIntro: '',
        coursePic: '',
        courseVideo: ''
      },
      uploadData: {},
      fileList: [],
      newFile: new FormData()
    }
  },
  components: {
    AddOrUpdate
  },
  activated () {
    this.getDataList()
  },
  methods: {
    suresubmit () {
      const that = this
      that.$confirm('确定交卷?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        that.submit()
      })
    },
    submit () {
      this.dialogFormVisibleExam = false
      console.log('交卷', this.testPaper)
      // this.testPaper.uid =
      this.$http({
        url: this.$http.adornUrl(`/exam/answerpaper/save`),
        method: 'post',
        data: this.testPaper
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
    },
    // submit (formName) {
    //   this.$refs[formName].validate((valid) => {
    //     if (!valid) {
    //       console.log('form1:', this.testPaper)
    //       return false
    //     } else {
    //       console.log('form2:', this.testPaper)
    //       const that = this
    //       if (that.testPaper.id === '') {
    //         console.log('执行添加试卷')
    //         this.$http({
    //           url: this.$http.adornUrl(`/exam/paper/save`),
    //           method: 'post',
    //           data: this.testPaper
    //         }).then(({data}) => {
    //           if (data && data.code === 0) {
    //             this.$message({
    //               message: '操作成功',
    //               type: 'success',
    //               duration: 1500,
    //               onClose: () => {
    //                 this.visible = false
    //                 this.$emit('refreshDataList')
    //               }
    //             })
    //           } else {
    //             this.$message.error(data.msg)
    //           }
    //         })
    //       }
    //     }
    //   })
    // },

    Upload (question) {
      console.log('question', question)
      const newData = this.newFile
      // newData.append('questionId', question.id)
      this.$http({
        url: this.$http.adornUrl('/exam/exam/upload'),
        method: 'post',
        data: newData,
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(({data}) => {
        if (data && data.code === 0) {
          console.log('上传成功', data.result)
          question.imageUrl = data.result
        } else {
        }
      })
    },
    BeforeUpload (file) {
      if (file) {
        this.newFile.set('file', file)
        this.uploadData.id = 110
        console.log('BeforeUpload', this.newFile.get('file'))
      } else {
        return false
      }
    },
    submitUpload () {
      this.$refs.uploadPic.submit()
    },
    handleSuccess (response, file, fileList) {
      console.log('handleSuccess111', file, 'fileList', fileList)
    },
    handlePreview (file) {
      console.log('handlePreview', file)
    },
    handleRemove (file, fileList) {
      console.log('handleRemove', file, 'fileList', fileList)
    },
    handleExceed (file, fileList) {
      this.$message({
        message: '只能选择一个文件上传',
        type: 'warning'
      })
    },
    surejoin () {
      // this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/exam/paper/detail/' + this.examination.paperId),
        method: 'get',
        params: this.$http.adornParams({
          'action': 'joinExam'
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.testPaper = data.data
          this.questions = data.data.questionList
          this.testPaper.paperId = data.data.id
          this.dialogFormVisibleExam = true
        } else {
          this.$message.error(data.msg)
        }
        this.dialogFormVisible = false
      })
    },

    joinExam (row) {
      this.examination.id = row.id
      this.examination.name = row.name
      this.examination.totalTime = row.totalTime
      this.examination.totalScore = row.totalScore
      this.examination.paperId = row.paperId
      this.examination.paperName = row.paperName
      this.dialogFormVisible = true
    },
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/exam/exam/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'key': this.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list
          this.totalPage = data.page.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    // 新增 / 修改
    addOrUpdateHandle (id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 删除
    deleteHandle (id) {
      var ids = id ? [id] : this.dataListSelections.map(item => {
        return item.id
      })
      this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/exam/exam/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    }
  }
}
</script>
<style>
.questionClass {
  margin-top: 1px;
  margin-left: 20%;
  margin-right: 20%;
  background: white;
}

.questionClass2 {
  margin-left: 20px;
  margin-right: 20px;
  margin-top: 20px;
  line-height: 30px;
  border-bottom: 1px solid black;
}

.testPaper .el-dialog {
  background: #eeeeee;
}

.testPaper .el-dialog__header {
  display: none;
}

.testPaper .dj-dialog-content {
  padding: 0;
  overflow: unset;
}

.testPaperHeader {
  text-align: center;
  line-height: 35px;
  border-bottom: 1px solid black;
  margin-left: 20px;
  margin-right: 20px;
}
</style>
