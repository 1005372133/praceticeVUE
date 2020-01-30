<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <!-- <el-button v-if="isAuth('generator:dadilyreport:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('generator:dadilyreport:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
        <el-button  type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button  type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
        <el-button  type="primary" @click="uploadVisible = true">项目上传</el-button>
        <el-button  type="primary" @click="taskVisible = true">实习任务</el-button>
         <el-button  type="primary" @click="scoreVisible = true">查看成绩</el-button>
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
        <template slot-scope="scope">
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




<el-dialog
  title="上传"
  :visible.sync="uploadVisible"
  width="22%">
    <el-upload
  class="upload-demo"
  drag
  action="http://localhost:8080/renren-fast/generator/dadilyreport/upload"
  :headers="myHeaders"
  multiple>
  <i class="el-icon-upload"></i>
  <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
</el-upload>
 <span slot="footer" class="dialog-footer">
    <el-button @click="uploadVisible = false">取 消</el-button>
  </span>
</el-dialog>


<el-dialog
  title="实习任务"
  :visible.sync="taskVisible"
  width="60%">


 <el-table
      :data="taskList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="任务名称">
      </el-table-column>
      <el-table-column
        prop="contest"
        header-align="center"
        align="center"
        label="正文">
      </el-table-column>
      <el-table-column
        prop="createuser"
        header-align="center"
        align="center"
        label="创建人">
      </el-table-column>
      <el-table-column
        prop="begintime"
        header-align="center"
        align="center"
        label="开始时间">
      </el-table-column>
      <el-table-column
        prop="endtime"
        header-align="center"
        align="center"
        label="结束时间">
      </el-table-column>
      <el-table-column
        prop="createtime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="flag"
        header-align="center"
        align="center"
        label="任务状态">
      </el-table-column>
    </el-table>




 <span slot="footer" class="dialog-footer">
    <el-button @click="taskVisible = false">取 消</el-button>
  </span>
</el-dialog>


<el-dialog
  title="成绩"
  :visible.sync="scoreVisible"
  width="30%">
  期中成绩：{{this.qzscoure}}
  <br>
  期中评价：{{this.qzcomment}}
  <br>
  期末成绩：{{this.qmcsoure}}
  <br>
  期末评价：{{this.qmcomment}}
</el-dialog>

  </div>
</template>

<script>
  import AddOrUpdate from './dadilyreport-add-or-update'
  import Vue from 'vue'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        uploadVisible: false,
        taskVisible: false,
        scoreVisible: false,
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        taskList: [],
        myHeaders: {token: Vue.cookie.get('token')},
        qzscoure: '',
        qzcomment: '',
        qmcsoure: '',
        qmcomment: ''
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
      this.gettaskList()
      this.getScoreDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/dadilyreport/list'),
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
      // 获取登陆人实习任务
      gettaskList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/dadilyreport/getTask'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.taskList = data.page.list
          } else {
            this.taskList = []
          }
          this.taskVisible = false
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
            url: this.$http.adornUrl('/generator/dadilyreport/delete'),
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
      // 获取数据列表
      getScoreDataList () {
        this.$nextTick(() => {
          this.$http({
            url: this.$http.adornUrl(`/generator/task/queryAllStuNameScoreStu`),
            method: 'post',
            params: this.$http.adornParams(
            )
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.qzscoure = data.page[0].qzscoure
              this.qzcomment = data.page[0].qzcomment
              this.qmcsoure = data.page[0].qmcsoure
              this.qmcomment = data.page[0].qmcomment
            }
          })
         // }
        })
      }
    }
  }
</script>
