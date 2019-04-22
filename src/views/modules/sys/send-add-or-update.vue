<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '发送邮件'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="标题" prop="title">
        <el-input v-model="dataForm.title" placeholder="标题"></el-input>
      </el-form-item>
      <el-form-item label="内容" prop="content" >
        <script :id="ueId" class="ueditor-box" type="text/plain" style="width: 100%; height: 260px;" v-model="dataForm.content">123</script>
      </el-form-item>
      <el-form-item label="附加文件" prop="fileList">
        <el-input v-model="dataForm.fileList" placeholder="附加文件"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import ueditor from 'ueditor'
  export default {props: ['userId'],
    data: function () {
      return {
        ue: null,
        ueId: `J_ueditorBox_${new Date().getTime()}`,
        ueContent: '',
        visible: false,
        roleList: [],
        dataForm: {
          title: '',
          content: '',
          receivers: '',
          fileList: ''
        },
        dataRule: {
          title: [
            {required: true, message: '标题不能为空', trigger: 'blur'}
          ]
        },
        dataListSelections: []
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/sys/role/select'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.roleList = data && data.code === 0 ? data.list : []
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }).then(() => {
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/user/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userName = data.user.username
                this.dataForm.salt = data.user.salt
                this.dataForm.email = data.user.email
                this.dataForm.mobile = data.user.mobile
                this.dataForm.roleIdList = data.user.roleIdList
                this.dataForm.status = data.user.status
              }
            }).then(this.ue = ueditor.getEditor(this.ueId, {
              // serverUrl: '', // 服务器统一请求接口路径
              zIndex: 3000,
              // focus时自动清空初始化时的内容
              autoClearinitialContent: true,
              // 关闭字数统计
              wordCount: false,
              // 关闭elementPath
              elementPathEnabled: false
            }))
          }
        })
      },
      // 表单提交
      dataFormSubmit (id) {
        var userIds = id ? [id] : this.dataListSelections.map(item => {
          return item.userId
        })
        this.$http({
          url: this.$http.adornUrl(`/sys/send/send`),
          method: 'post',
          data: this.$http.adornData({
            'title': this.dataForm.title,
            'content': this.dataForm.content,
            'receivers': this.dataForm.id,
            'fileList': this.dataForm.fileList
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
    }
  }
</script>
