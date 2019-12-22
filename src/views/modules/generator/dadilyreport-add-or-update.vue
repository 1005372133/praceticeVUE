<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="内容" prop="content">
      <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" v-model="dataForm.content"  placeholder="请输入内容">
      </el-input>
    </el-form-item>
    <!-- <el-form-item label="得分" prop="score">
      <el-input v-model="dataForm.score" placeholder="得分"></el-input>
    </el-form-item>
    <el-form-item label="意见" prop="opinion">
      <el-input v-model="dataForm.opinion" placeholder="意见"></el-input>
    </el-form-item> -->
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
          content: '',
          time: '',
          user: '',
          score: '',
          opinion: ''
        },
        dataRule: {
          content: [
            { required: true, message: '不能为空', trigger: 'blur' }
          // ],
          // score: [
          //   { required: true, message: '不能为空', trigger: 'blur' }
          // ],
          // opinion: [
          //   { required: true, message: '不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/dadilyreport/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.content = data.dadilyreport.content
                this.dataForm.user = data.dadilyreport.user
                this.dataForm.score = data.dadilyreport.score
                this.dataForm.opinion = data.dadilyreport.opinion
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
              url: this.$http.adornUrl(`/generator/dadilyreport/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'content': this.dataForm.content
                // 'score': this.dataForm.score,
                // 'opinion': this.dataForm.opinion
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
