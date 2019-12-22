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
       <el-date-picker v-model="dataForm.begintime" type="date" placeholder="开始时间"  format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
    </el-form-item>
    <el-form-item label="结束时间" prop="endtime">
       <el-date-picker v-model="dataForm.endtime" type="date" placeholder="结束时间"  format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
    </el-form-item>
    <el-form-item label="实习学生" prop="getuser">
      <el-input v-model="dataForm.getuser" placeholder="实习学生"></el-input>
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
          createuser: '',
          begintime: '',
          endtime: '',
          createtime: '',
          getuser: '',
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
              url: this.$http.adornUrl(`/generator/task/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.task.name
                this.dataForm.begintime = data.task.begintime
                this.dataForm.endtime = data.task.endtime
                this.dataForm.getuser = data.task.getuser
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
                'begintime': this.dataForm.begintime.toString,
                'endtime': this.dataForm.endtime.toString,
                'getuser': this.dataForm.getuser
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
