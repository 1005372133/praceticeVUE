<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <!-- <el-button v-if="isAuth('generator:suggest:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('generator:suggest:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
         <!-- <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button> -->
        <!-- <el-button  type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
        <el-button type="text" @click="dialogVisible = true">日常日报评分</el-button>
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
        prop="user_id"
        header-align="center"
        align="center"
        label="id">
      </el-table-column>
      <el-table-column
        prop="username"
        header-align="center"
        align="center"
        label="学生名称">
      </el-table-column>
      <el-table-column
        prop="mobile"
        header-align="center"
        align="center"
        label="手机号">
      </el-table-column>
      <el-table-column
        prop="qzscoure"
        header-align="center"
        align="center"
        label="中期分数">
      </el-table-column>
      <el-table-column
        prop="qmcsoure"
        header-align="center"
        align="center"
        label="期末分数">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id,0)">期中成绩</el-button>
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id,1)">期末成绩</el-button>
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

    <el-dialog
  title="提示"
  :visible.sync="dialogVisible"
  width="70%"
  :before-close="handleClose">
  <el-table
      :data="reportList"
      border
      v-loading="reportListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">

      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="id">
      </el-table-column>
      <el-table-column
        prop="content"
        header-align="center"
        align="center"
        label="内容">
      </el-table-column>
      <el-table-column
        prop="time"
        header-align="center"
        align="center"
        label="发布时间">
      </el-table-column>
      <el-table-column
        prop="user"
        header-align="center"
        align="center"
        label="发布人">
      </el-table-column>
      <el-table-column
        prop="score"
        header-align="center"
        align="center"
        label="得分">
      </el-table-column>
      <el-table-column
        prop="opinion"
        header-align="center"
        align="center"
        label="意见">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="commentscope">
          <el-button type="text" size="small" @click="comment(commentscope.row.id)">评论</el-button>
        </template>
</el-table-column>
  </el-table>
</el-dialog>
  <!-- 评论 模态-->
    
     <el-dialog
  title="提示"
  :visible.sync="commentdialogVisible"
  width="70%"
  :before-close="handleClose">
  评分：
<el-input v-model="score" placeholder="请输入内容"></el-input>
意见：
<el-input v-model="opinion" placeholder="请输入内容"></el-input>
 <span slot="footer" class="dialog-footer">
    <el-button @click="commentdialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="commentFormSubmit()">确 定</el-button>
  </span>
</el-dialog>
<!-- <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update> -->

  </div>
</template>

<script>
  import AddOrUpdate from './suggest-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        commentdataForm: {
          reportkey: ''
        },
        commentId: '',
        score: '',
        opinion: '',
        dataList: [],
        reportList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        reportListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        commentVisible: false,
        dialogVisible: false,
        commentdialogVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
      this.getReportList()
    },
    methods: {
      handleClose (done) {
        this.$confirm('确认关闭？')
          .then(_ => {
            done()
          })
          .catch(_ => {})
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/task/queryAllStuNameScore'),
          method: 'post',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page
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
      addOrUpdateHandle (id, flag) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id, flag)
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
            url: this.$http.adornUrl('/generator/suggest/delete'),
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
      // 获取日报数据
      getReportList () {
        this.reportListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/dadilyreport/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.reportkey
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.reportList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.reportList = []
            this.totalPage = 0
          }
          this.reportListLoading = false
        })
      },
      // 打分
      comment (id) {
        this.commentdialogVisible = true
        this.commentId = id
        // this.$nextTick(() => {
        //   this.$refs.addOrUpdate.init(id)
        // })
      },
      // 评论
      commentFormSubmit () {
        this.$http({
          url: this.$http.adornUrl('/generator/dadilyreport/update'),
          method: 'post',
          data: this.$http.adornData({
            'id': this.commentId,
            // 'content': this.dataForm.content
            'score': this.score,
            'opinion': this.opinion
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.commentdialogVisible = false
                this.getReportList()
                this.$emit('refreshDataList')
                this.score = ''
                this.opinion = ''
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      }
    }
  }
</script>
