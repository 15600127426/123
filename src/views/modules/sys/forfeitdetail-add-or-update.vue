<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="具体事情" prop="event">
      <el-input v-model="dataForm.event" placeholder="具体事情"></el-input>
    </el-form-item>
    <el-form-item label="数目" prop="amount">
      <el-input v-model="dataForm.amount" placeholder="数目"></el-input>
    </el-form-item>
    <el-form-item label="人" prop="nickname">
      <el-input v-model="dataForm.nickname" placeholder="人"></el-input>
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
          event: '',
          amount: '',
          nickname: ''
        },
        dataRule: {
          event: [
            { required: true, message: '具体事情不能为空', trigger: 'blur' }
          ],
          amount: [
            { required: true, message: '数目不能为空', trigger: 'blur' }
          ],
          nickname: [
            { required: true, message: '人不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/sys/forfeitdetail/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.event = data.forfeitdetail.event
                this.dataForm.amount = data.forfeitdetail.amount
                this.dataForm.nickname = data.forfeitdetail.nickname
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
              url: this.$http.adornUrl(`/sys/forfeitdetail/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'event': this.dataForm.event,
                'amount': this.dataForm.amount,
                'nickname': this.dataForm.nickname
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
