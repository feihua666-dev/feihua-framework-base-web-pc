<template>

  <div class="wrapper">
    <el-container>
      <el-main>
        <el-collapse value="1">
          <el-collapse-item title="查询条件" name="1">
            <el-form ref="searchForm" :model="searchFormModel" :inline="true" size="small">
              <el-form-item label="标题">
                <el-input  v-model="searchFormModel.title"></el-input>
              </el-form-item>
              <el-form-item label="消息分类">
                <self-dict-select v-model="searchFormModel.msgType" type="message_type"></self-dict-select>
              </el-form-item>
              <el-form-item label="消息紧急性">
                <self-dict-select v-model="searchFormModel.msgLevel" type="message_level"></self-dict-select>
              </el-form-item>
              <el-form-item label="是否已读">
                <self-dict-select v-model="searchFormModel.isRead" type="yes_no"></self-dict-select>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="searchBtnClick">查询</el-button>
              </el-form-item>
            </el-form>
          </el-collapse-item>
        </el-collapse>
        <self-table :columns="columns" :tableData="tableData" :page="page" :table-loading="tableLoading" v-on:pageSizeChange="pageSizeChange" v-on:pageNoChange="pageNoChange"></self-table>
      </el-main>
    </el-container>

  </div>
</template>

<script>
  import SelfPage from '@/components/SelfPage.vue'
  import SelfTable from '@/components/SelfTable.vue'
  import SelfDictSelect from '@/components/SelfDictSelect.vue'
  export default {
    name: 'MyMessage',
    components: {
      SelfDictSelect,
      SelfTable,
      SelfPage
    },
    data () {
      return {
        columns: [
          {
            name: 'title',
            label: '标题'
          },
          {
            name: 'profile',
            label: '简介'
          },
          {
            name: 'content',
            label: '内容'
          },
          {
            name: 'msgType',
            label: '消息分类',
            dict: 'message_type'
          },
          {
            name: 'msgLevel',
            label: '紧急度',
            dict: 'message_level'
          },
          {
            name: 'isRead',
            label: '是否已读',
            dict: 'yes_no'
          },
          {
            label: '操作',
            width: '300',
            buttons: [
              {
                label: '查看',
                click: this.viewTableRowClick
              },
              {
                label: this.markTableRowLabel,
                click: this.markTableRowClick
              }
            ]
          }
        ],
        page: {
          dataNum: 0
        },
        // 表格数据
        tableData: [],
        tableLoading: false,
        // 搜索的查询条件
        searchFormModel: {
          title: '',
          msgLevel: '',
          msgType: '',
          isRead: '',
          pageable: true,
          pageNo: 1,
          pageSize: 10
        }
      }
    },
    mounted () {
      this.loadTableData(1)
    },
    methods: {
      // 查询按钮点击事件
      searchBtnClick () {
        this.loadTableData(1)
      },
      // 加载表格数据
      loadTableData (pageNo) {
        let self = this
        if (pageNo) {
          self.searchFormModel.pageNo = pageNo
        }
        self.tableLoading = true
        this.$http.get('/base/message/currentuser/messages', self.searchFormModel)
          .then(function (response) {
            let content = response.data.data.content
            self.tableData = content
            self.page.dataNum = response.data.data.page.dataNum
            self.tableLoading = false
          })
          .catch(function (error) {
            if (error.response.status === 404) {
              self.tableData = []
              self.page.dataNum = 0
            }
            self.tableLoading = false
          })
      },
      // 页面大小改变重新查询数据
      pageSizeChange (val) {
        this.searchFormModel.pageSize = val
        this.searchBtnClick()
      },
      // 页码改变加载对应页码数据
      pageNoChange (val) {
        this.loadTableData(val)
      },
      // 查看消息
      viewTableRowClick (index, row) {
        let self = this
        this.$http.post('/base/message/' + row.id + '/currentuser')
          .then(function (response) {
            let content = response.data.data.content
            self.$alert(content.content, content.title, {
              confirmButtonText: '确定'
            }
            )
            if (row.isRead === 'N') {
              // 重新加载数据
              self.loadTableData()
            }
          })
          .catch(function (error) {
            if (error.response.status === 404) {
              let response = error.response
              let content = response.data.data.content
              self.$alert(content.content, content.title, {
                confirmButtonText: '确定'}
              )
              if (row.isRead === 'N') {
                // 重新加载数据
                self.loadTableData()
              }
            }
          })
      },
      markTableRowLabel (index, row) {
        if (row.isRead === 'N') {
          return '标记为已读'
        } else {
          return ''
        }
      },
      // 标记为已读
      markTableRowClick (index, row) {
        let self = this
        this.$http.post('/base/message/' + row.id + '/currentuser', {markOnly: true})
          .then(function (response) {
            self.$message.success('标记已读成功')
            // 重新加载数据
            self.loadTableData()
          })
          .catch(function (error) {
            if (error.response.status === 404) {
              self.$message.error('标记已读失败，请刷新数据再试')
            }
          })
      }
    },
    watch: {
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .wrapper .el-collapse{
    padding: 0 10px;
  }
.el-main{
  padding:0;
}
.el-aside{
  border-right: 1px solid #e6ebf5;
}
.wrapper,.el-container{
  height:100%;
}
</style>
<style>
.el-collapse-item__arrow {
  /* 由于用了rotate 这个东西不是个正方形所以改变角度的时候会出现滚动条 */
  margin-right: 20px;
}
</style>
