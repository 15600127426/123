<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="名字" prop="userName">
      <el-input v-model="dataForm.userName" placeholder="名字"></el-input>
    </el-form-item>
    <el-form-item label="外号" prop="nickname">
      <el-input v-model="dataForm.nickname" placeholder="外号"></el-input>
    </el-form-item>
    <el-form-item label="入职时间" prop="joinTime">
      <el-input v-model="dataForm.joinTime" placeholder="入职时间"></el-input>
    </el-form-item>
    <el-form-item label="来自" prop="province">
      <el-input v-model="dataForm.province" placeholder="来自"></el-input>
    </el-form-item>
    <el-form-item label="赔付" prop="needPaid">
      <el-input v-model="dataForm.needPaid" placeholder="赔付"></el-input>
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
          userName: '',
          nickname: '',
          joinTime: '',
          province: '',
          needPaid: ''
        },
        dataRule: {
          userName: [
            { required: true, message: '名字不能为空', trigger: 'blur' }
          ],
          nickname: [
            { required: true, message: '外号不能为空', trigger: 'blur' }
          ],
          joinTime: [
            { required: true, message: '入职时间不能为空', trigger: 'blur' }
          ],
          province: [
            { required: true, message: '来自不能为空', trigger: 'blur' }
          ],
          needPaid: [
            { required: true, message: '赔付不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/sys/paemployee/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userName = data.paemployee.userName
                this.dataForm.nickname = data.paemployee.nickname
                this.dataForm.joinTime = data.paemployee.joinTime
                this.dataForm.province = data.paemployee.province
                this.dataForm.needPaid = data.paemployee.needPaid
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
              url: this.$http.adornUrl(`/sys/paemployee/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userName': this.dataForm.userName,
                'nickname': this.dataForm.nickname,
                'joinTime': this.dataForm.joinTime,
                'province': this.dataForm.province,
                'needPaid': this.dataForm.needPaid
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
