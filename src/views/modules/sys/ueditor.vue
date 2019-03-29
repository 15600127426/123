<template>
  <div class="mod-demo-ueditor">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()"></el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <!--<el-table-column
      prop="id"
      header-align="center"
      align="center"
      width="80"
      label="ID">
    </el-table-column>
    <el-table-column
      prop="user_id"
      header-align="center"
      align="center"
      label="用户id">
    </el-table-column>
    <el-table-column
      prop="avatar"
      header-align="center"
      align="center"
      width="180"
      label="昵称">
    </el-table-column> -->
      <el-table-column
      prop="title"
      header-align="center"
      align="center"
      label="标题">
    </el-table-column>
      <el-table-column
        prop="detail"
        header-align="center"
        align="center"
        label="细节">
      </el-table-column>
      <el-table-column
      prop="createTime"
      header-align="center"
      align="center"
      width="180"
      label="发表时间">
    <!--<el-input v-model="input" placeholder="这里是标题" clearable></el-input>
    <<el-input
    type="textarea"
    :autosize="{minRows: 3, maxRows: 15}"
    placeholder="这里是内容"
    v-model="textarea">
  </el-input>-->
    </el-table-column></el-table>
    <el-button @click="getDataList()">刷新</el-button>
    <el-button v-if="isAuth('generator:advice:save')" type="primary" @click="addOrUpdateHandle()">提交</el-button>
      <el-alert
    title="提示："
    type="warning"
    :closable="false">
    <div slot-scope="description">
      <p class="el-alert__description">1.每周只能提一次</p>
    </div>
  </el-alert>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>

</template>

   <script>
  import ueditor from 'ueditor'
  import AddOrUpdate from './ueditor-add-or-update'
  export default {
    data: function () {
      return {
        dataForm: {
          title: '',
          detail: '',
          user_id: '',
          avatar: '',
          createTime: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        dataRule: {
          title: [
            {required: true, message: '名称不能为空', trigger: 'blur'}],
          detail: [
            {required: true, message: '名称不能为空', trigger: 'blur'}],
          user_id: [
            {required: true, message: '名称不能为空', trigger: 'blur'}
          ],
          tempKey: -666666 // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
        }
      }
  },
    components: {
      AddOrUpdate
    },
    methods: {
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/advice/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'title': this.dataForm.title
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
          // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
          // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
          // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      getContent () {
        this.dialogVisible = true
        this.ue.ready(() => {
          this.ueContent = this.ue.getContent()
        })
      }, // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/generator/advice/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      }
    }
  }
    </script>

<style lang="scss">
  .mod-demo-ueditor {
    position: relative;
    z-index: 510;
    > .el-alert {
      margin-bottom: 10px;
    }
  }
</style>
