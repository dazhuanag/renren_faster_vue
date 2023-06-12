<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
<!--        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>-->
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">刷新</el-button>
<!--        <el-button v-if="isAuth('exam:answerpaper:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>-->
<!--        <el-button v-if="isAuth('exam:answerpaper:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>-->
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
<!--      <el-table-column-->
<!--        prop="paperId"-->
<!--        header-align="center"-->
<!--        align="center"-->
<!--        label="ID">-->
<!--      </el-table-column>-->
      <el-table-column
        prop="userName"
        header-align="center"
        align="center"
        label="学生姓名">
      </el-table-column>
      <el-table-column
        prop="paperName"
        header-align="center"
        align="center"
        label="试卷名称">
      </el-table-column>
      <el-table-column
        prop="paperTotalScore"
        header-align="center"
        align="center"
        label="试卷总分">
      </el-table-column>
      <el-table-column
        prop="paperTotalTime"
        header-align="center"
        align="center"
        label="考试时间">
      </el-table-column>
      <el-table-column
        prop="userTotalScore"
        header-align="center"
        align="center"
        label="考试得分">
      </el-table-column>
      <el-table-column
        prop="examTime"
        header-align="center"
        align="center"
        label="参加考试时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
<!--          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">预览</el-button>-->
          <el-button type="text" size="small" @click="look(scope.row)">预览</el-button>
          <el-button type="text" size="small" @click="remove(scope.row.paperId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
<!--    <el-pagination-->
<!--      @size-change="sizeChangeHandle"-->
<!--      @current-change="currentChangeHandle"-->
<!--      :current-page="pageIndex"-->
<!--      :page-sizes="[10, 20, 50, 100]"-->
<!--      :page-size="pageSize"-->
<!--      :total="totalPage"-->
<!--      layout="total, sizes, prev, pager, next, jumper">-->
<!--    </el-pagination>-->
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <div class="testPaper">
      <el-dialog
        :visible.sync="dialogFormVisibleTestPaper"
        :fullscreen="true">
        <div class="questionClass">
          <div class="testPaperHeader">
            <div style="font-size:20px;">{{testpaper.name}}
              <el-button type="primary" @click="dialogFormVisibleTestPaper=false" style="float:right;margin-top: 10px;margin-left:20px;">关闭
              </el-button>
            </div>
            <div>考试时间：{{testpaper.totalTime}}分钟
              &nbsp;&nbsp;&nbsp;总分：{{testpaper.totalScore}}
              &nbsp;&nbsp;&nbsp;考试得分：{{testpaper.userTotalScore}}
            </div>
          </div>
          <div v-for="(question,index) in testpaper.questionList" class="questionClass2">
            {{index+1}}.{{question.content}} (总分:{{question.score}})
            <div v-if="question.type=='简答题'" style="margin-top:10px;margin-bottom: 20px;">
              <div>
                <el-input v-model="question.answer" disabled type="textarea" >
                </el-input>
<!--                <el-image :src="question.imageUrl" fit="contain"></el-image>-->
                <img :src="question.imageUrl"  style="width: 80%; height:38%" >
                <div>
                  <h3>得分:
                    <input v-model="question.getScore" disabled>
                    </input>
                  </h3>
                </div>
            </div>
            </div>
          </div>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
  import AddOrUpdate from './answerpaper-add-or-update'
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
        dialogFormVisibleTestPaper: false,
        testpaper: {}
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/exam/answerpaper/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.data
          } else {
            this.dataList = []
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
            url: this.$http.adornUrl('/exam/answerpaper/delete'),
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
      },
      // 删除
      remove (id) {
        this.$confirm(`确定对[id=${id}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/exam/answerpaper/remove/' + id),
            method: 'post'
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
      },
      look (row) {
        this.dialogFormVisibleTestPaper = true
        this.$http({
          url: this.$http.adornUrl('/exam/answerpaper/detail/' + row.paperId),
          method: 'get',
          params: this.$http.adornParams({
            'userId': row.userId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            // this.dataList = data.page.list
            this.testpaper = data.data
          }
          this.dataListLoading = false
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
