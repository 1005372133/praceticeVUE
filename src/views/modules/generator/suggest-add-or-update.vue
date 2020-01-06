<template>
  <el-dialog
    :title="dataForm.flag==0 ? '期中评定' : '期末评定'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item v-show="dataForm.flag==0" label="期中成绩" prop="committime">
      <el-input v-model="dataForm.qzscoure" placeholder=""></el-input>
    </el-form-item>
    <el-form-item v-show="dataForm.flag==0" label="期中评价" prop="content">
      <el-input v-model="dataForm.qzcomment" placeholder=""></el-input>
    </el-form-item>
    <el-form-item v-show="dataForm.flag==1" label="期末成绩" prop="committime">
      <el-input v-model="dataForm.qmcsoure" placeholder=""></el-input>
    </el-form-item>
    <el-form-item v-show="dataForm.flag==1" label="期末评价" prop="content">
      <el-input v-model="dataForm.qmcomment" placeholder=""></el-input>
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
          flag: '',
          id: 0,
          qzscoure: '',
          qzcomment: '',
          qmcsoure: '',
          qmcomment: '',
          userid: ''
        },
        dataRule: {
          qzscoure: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          qzcomment: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          qmcsoure: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          qmcomment: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id, flag) {
        this.dataForm.flag = flag
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          // if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/generator/suggest/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.time = data.suggest.time
              this.dataForm.createuser = data.suggest.createuser
              this.dataForm.foruser = data.suggest.foruser
              this.dataForm.content = data.suggest.content
            }
          })
         // }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/suggest/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'time': this.dataForm.time,
                'createuser': this.dataForm.createuser,
                'foruser': this.dataForm.foruser,
                'content': this.dataForm.content
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
