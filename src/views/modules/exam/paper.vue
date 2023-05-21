<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
<!--        <el-button v-if="isAuth('exam:paper:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>-->
        <el-button v-if="isAuth('exam:paper:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
        label="试卷名称">
      </el-table-column>
      <el-table-column
        prop="totalScore"
        header-align="center"
        align="center"
        label="试卷总数">
      </el-table-column>
      <el-table-column
        prop="totalTime"
        header-align="center"
        align="center"
        label="试卷总时长(min)">
      </el-table-column>
      <el-table-column
        prop="majorId"
        header-align="center"
        align="center"
        label="学院">
      </el-table-column>
      <el-table-column
        prop="courseId"
        header-align="center"
        align="center"
        label="所属课程">
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
          <el-button type="text" size="small" @click="look(scope.row)">预览</el-button>
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
            </div>
          </div>
          <div v-for="(question,index) in testpaper.questionList" class="questionClass2">
            {{index+1}}.{{question.content}} ({{question.score}}分)
            <div style="margin-top:10px;">
              <el-image v-if="question.imgurl" :src="getImg(question.imgurl)"
                        :preview-src-list="getPreviewImg(question.imgurl)" style="width: 200px; height: 200px">
              </el-image>
            </div>
            <div v-if="question.type=='简答题'" style="margin-top:10px;margin-bottom: 20px;">
              <div>
                <el-input v-model="question.reply" type="textarea" :rows="5">
                </el-input>
              </div>
            </div>
          </div>
        </div>
      </el-dialog>
    </div>

  </div>
</template>

<script>
  import AddOrUpdate from './paper-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        testpaper: {},
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        dialogFormVisibleTestPaper: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 预览
      // look(row){
      //   this.dialogFormVisibleTestPaper=true;
      //   const that = this;
      //   var param = new URLSearchParams();
      //   param.append("id", row.id);
      //   that.$http
      //     .get('/examinationManage/getTestPaperById', {
      //       params: param
      //     })
      //     .then(function (response) {
      //       if (response.data.code == 200) {
      //         that.testpaper = response.data.data;
      //       }
      //     })
      // },
      // 获取数据列表
      look (row) {
        this.dialogFormVisibleTestPaper = true
        this.$http({
          url: this.$http.adornUrl('/exam/paper/detail/' + row.id),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            // this.dataList = data.page.list
            this.testpaper = data.data
          }
          this.dataListLoading = false
        })
      },

      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/exam/paper/list'),
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
            url: this.$http.adornUrl('/exam/paper/delete'),
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
