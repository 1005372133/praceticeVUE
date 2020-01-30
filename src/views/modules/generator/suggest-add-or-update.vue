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
        this.visible = true
        this.dataForm.flag = flag
        this.dataForm.userid = id
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          // if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/generator/task/queryAllStuNameScore`),
            method: 'post',
            params: this.$http.adornParams({
              'userId': id
            }
            )
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.qzscoure = data.page[0].qzscoure
              this.dataForm.qzcomment = data.page[0].qzcomment
              this.dataForm.qmcsoure = data.page[0].qmcsoure
              this.dataForm.qmcomment = data.page[0].qmcomment
              this.dataForm.id = data.page[0].id
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
              url: this.$http.adornUrl(`/generator/score/updateScore`),
              method: 'post',
              data: this.$http.adornData({
                'qzscoure': this.dataForm.qzscoure,
                'qzcomment': this.dataForm.qzcomment,
                'qmcsoure': this.dataForm.qmcsoure,
                'qmcomment': this.dataForm.qmcomment,
                'userid': this.dataForm.userid,
                'id': this.dataForm.id
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
