<template>
  <el-dialog
    :title="!dataForm.id ? '发表' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible" @close="kcDialog = false">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="标题" prop="title">
        <el-input v-model="dataForm.title" placeholder="标题" value="title"></el-input>
      </el-form-item>
      <el-form-item label="内容" prop="detail">
        <el-input v-model="dataForm.detail" placeholder="内容" value="detail" type="textarea" :autosize="{ minRows: 3, maxRows: 8}"></el-input>
      </el-form-item>
      <!--<el-form-item label="用户id" prop="user_id">
        <el-input v-model="dataForm.user_id" placeholder="用户id"></el-input>
      </el-form-item>
      <el-form-item label="发表时间" prop="creat_time">
        <el-input v-model="dataForm.creat_time" placeholder="发表时间"></el-input>
      </el-form-item>
      <el-form-item label="昵称" prop="avatar">
        <el-input v-model="dataForm.avatar" placeholder="昵称"></el-input>
      </el-form-item>-->

    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { treeDataTranslate } from '@/utils'
  export default {
    data () {
      return {
        visible: false,
        menuList: [],
        menuListTreeProps: {
          label: 'name',
          children: 'children'
        },
        dataForm: {
          title: '',
          detail: '',
          user_id: '',
          avatar: '',
          createTime: ''
        },
        dataRule: {
          title: [
            {required: true, message: '名称不能为空', trigger: 'blur'}],
          detail: [
            {required: true, message: '名称不能为空', trigger: 'blur'}]
        },
        tempKey: -666666 // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
      }
    },
    methods: {
      init: function (id) {
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/generator/advice/list'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.menuList = treeDataTranslate(data, 'menuId')
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
            this.$refs.menuListTree.setCheckedKeys([])
          })
        }).then(() => {
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/advice/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.user_id = data.role.user_id
                var idx = data.role.menuIdList.indexOf(this.tempKey)
                if (idx !== -1) {
                  data.role.menuIdList.splice(idx, data.role.menuIdList.length - idx)
                }
                this.$refs.menuListTree.setCheckedKeys(data.role.menuIdList)
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
              url: this.$http.adornUrl(`/generator/advice/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'title': this.dataForm.title,
                'detail': this.dataForm.detail,
                'createTime': new Date(),
                'user_id': this.dataForm.user_id,
                'avatar': this.dataForm.avatar
                // 'menuIdList': [].concat(this.$refs.menuListTree.getCheckedKeys(), [this.tempKey], this.$refs.menuListTree.getHalfCheckedKeys())
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

<style scoped>

</style>
