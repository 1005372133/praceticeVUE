<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="实习名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="实习名称"></el-input>
    </el-form-item>
    <el-form-item label="开始时间" prop="begintime">
       <el-date-picker v-model="dataForm.begintime" type="date" placeholder="开始时间"  format="yyyy-MM-dd"></el-date-picker>
    </el-form-item>
    <el-form-item label="结束时间" prop="endtime">
       <el-date-picker v-model="dataForm.endtime" type="date" placeholder="结束时间"  format="yyyy-MM-dd"></el-date-picker>
    </el-form-item>
    <el-form-item label="内容" prop="contest">
        <el-input v-model="dataForm.contest" placeholder="实习名称"></el-input>
    </el-form-item>
    <el-form-item label="实习学生" prop="getuser">
      <!-- <el-input v-model="dataForm.getuser" placeholder="实习学生">
        <el-checkbox v-model="checked">备选项</el-checkbox>
      </el-input> -->


       <el-checkbox-group v-model="dataForm.getuser" @change="handleCheckedCitiesChange">
    <el-checkbox v-for="stu in AllStu" :label="stu.username" :key="stu.user_id">{{stu.username}}</el-checkbox>
  </el-checkbox-group>

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
        dataList: '',
        AllStu: [],
        dataForm: {
          id: 0,
          name: '',
          createuser: '',
          begintime: '',
          endtime: '',
          createtime: '',
          getuser: '',
          contest: '',
          flag: ''
        },
        dataRule: {
          name: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          begintime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          endtime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          getuser: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          contest: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    mounted () {
      this.getAllStu()
    },
    methods: {
      getAllStu () {
        this.$http({
          url: this.$http.adornUrl('/generator/task/queryAllStuName'),
          method: 'post',
          params: this.$http.adornParams()
        }).then(({data}) => {
          console.log(data.page)
          this.AllStu = data.page
        })
      },
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/task/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.task.name
                this.dataForm.begintime = data.task.begintime
                this.dataForm.endtime = data.task.endtime
                this.dataForm.getuser = data.task.getuser
                this.dataForm.contest = data.task.contest
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
              url: this.$http.adornUrl(`/generator/task/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'begintime': this.dataForm.begintime,
                'endtime': this.dataForm.endtime,
                'getuser': this.dataForm.getuser,
                'contest': this.dataForm.contest
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
      },
      handleCheckAllChange (val) {
        // this.checkedCities = val ? cityOptions : []
        // this.isIndeterminate = false
      },
      handleCheckedCitiesChange (value) {
        // let checkedCount = value.length
        // this.checkAll = checkedCount === this.cities.length
        // this.isIndeterminate = checkedCount > 0 && checkedCount < this.cities.length
      }
    }
  }
</script>
