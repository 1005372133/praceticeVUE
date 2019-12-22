<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="" prop="name">
      <el-input v-model="dataForm.name" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="createuser">
      <el-input v-model="dataForm.createuser" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="begintime">
      <el-input v-model="dataForm.begintime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="endtime">
      <el-input v-model="dataForm.endtime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="createtime">
      <el-input v-model="dataForm.createtime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="getuser">
      <el-input v-model="dataForm.getuser" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="flag">
      <el-input v-model="dataForm.flag" placeholder=""></el-input>
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
          createuser: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          begintime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          endtime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          createtime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          getuser: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          flag: [
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
                this.dataForm.createuser = data.task.createuser
                this.dataForm.begintime = data.task.begintime
                this.dataForm.endtime = data.task.endtime
                this.dataForm.createtime = data.task.createtime
                this.dataForm.getuser = data.task.getuser
                this.dataForm.flag = data.task.flag
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
                'createuser': this.dataForm.createuser,
                'begintime': this.dataForm.begintime,
                'endtime': this.dataForm.endtime,
                'createtime': this.dataForm.createtime,
                'getuser': this.dataForm.getuser,
                'flag': this.dataForm.flag
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
