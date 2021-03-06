<template>

  <div class="wrapper">
    <el-container>
      <el-aside width="200px">
        <el-scrollbar style="height: 100%;" wrapStyle="height:100%;overflow:auto;" >
          <role-tree ref="lefttree" v-on:nodeClick="treeNodeClick"></role-tree>
         </el-scrollbar>
      </el-aside>
      <el-main>
        <el-collapse value="1">
          <el-collapse-item title="查询条件" name="1">
            <el-form ref="searchForm" :model="searchFormModel" :inline="true" size="small">
              <el-form-item label="名称" prop="name">
                <el-input  v-model="searchFormModel.name"></el-input>
              </el-form-item>
              <el-form-item label="类型" prop="type">
                <self-dict-select v-model="searchFormModel.type" type="role_type"></self-dict-select>
              </el-form-item>
              <el-form-item label="是否禁用" prop="disabled">
                <self-dict-select v-model="searchFormModel.disabled" type="yes_no"></self-dict-select>
              </el-form-item>
              <el-form-item label="机构">
                <OfficeInputSelect ref="officeinput"  v-model="searchFormModel.dataOfficeId">
                </OfficeInputSelect>
              </el-form-item>
              <el-form-item label="父级">
                <RoleInputSelect ref="roleinput"  v-model="searchFormModel.parentId">
                </RoleInputSelect>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="searchBtnClick">查询</el-button>
                <el-button type="primary" @click="addTableRowClick">添加</el-button>
                <el-button @click="resetFormClick">重置</el-button>
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
  import RoleTree from './RoleTree.vue'
  import SelfPage from '@/components/SelfPage.vue'
  import SelfTable from '@/components/SelfTable.vue'
  import loadDataControl from '@/utils/storeLoadDataControlUtils.js'
  import SelfDictSelect from '@/components/SelfDictSelect.vue'
  import { getDictByValueSync } from '@/utils/dictUtils.js'
  import OfficeInputSelect from '@/views/office/OfficeInputSelect'
  import RoleInputSelect from '@/views/role/RoleInputSelect.vue'
  export default {
    name: 'Role',
    components: {
      SelfDictSelect,
      SelfTable,
      RoleTree,
      SelfPage,
      OfficeInputSelect,
      RoleInputSelect
    },
    data () {
      return {
        columns: [
          {
            name: 'name',
            label: '名称'
          },
          {
            name: 'code',
            label: '编码'
          },
          {
            name: 'type',
            label: '类型',
            formatter: this.typeFormatter
          },
          {
            name: 'disabled',
            label: '是否禁用',
            formatter: this.disabledFormatter
          },
          {
            name: 'parentId',
            label: '父级',
            formatter: this.dataParentFormatter
          },
          {
            name: 'dataOfficeId',
            label: '机构',
            formatter: this.dataOfficeFormatter
          },
          {
            label: '操作',
            width: '270',
            buttons: [
              {
                label: '编辑',
                click: this.editTableRowClick
              },
              {
                label: '绑定数据范围',
                click: this.bindDataScope
              },
              {
                label: '设置功能资源',
                click: this.setFunctionResource
              },
              {
                label: '删除',
                click: this.deleteTableRowClick
              }
            ]
          }
        ],
        page: {
          pageNo: 1,
          dataNum: 0
        },
        tableOffice: {},
        tableParent: {},
        // 表格数据
        tableData: [],
        tableLoading: false,
        // 搜索的查询条件
        searchFormModel: {
          name: '',
          code: '',
          type: '',
          disabled: '',
          parentId: '',
          dataOfficeId: '',
          includeOffice: true,
          includeParent: true,
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
      // 点击树节点事件
      treeNodeClick (data) {
        this.$refs.roleinput.setLabelName(data.name)
        this.searchFormModel.parentId = data.id
        this.searchBtnClick()
      },
      resetFormClick () {
        this.$refs.searchForm.resetFields()
        this.$refs.officeinput.setLabelName(null)
        this.$refs.roleinput.setLabelName(null)
        this.searchFormModel.parentId = null
        this.searchFormModel.dataOfficeId = null
      },
      // 查询按钮点击事件
      searchBtnClick () {
        this.loadTableData(1)
      },
      // 加载表格数据
      loadTableData (pageNo, pageNoChange) {
        let self = this
        if (pageNo > 0) {
          if (pageNoChange) {
            self.searchFormModel.pageNo = pageNo
          } else {
            if (self.page.pageNo !== pageNo) {
              self.page.pageNo = pageNo
              return
            }
          }
        }
        self.tableLoading = true
        this.$http.get('/base/roles', self.searchFormModel)
          .then(function (response) {
            let content = response.data.data.content
            self.tableOffice = response.data.data.office
            self.tableParent = response.data.data.parent
            self.tableData = content
            self.page.dataNum = response.data.data.page.dataNum
            self.tableLoading = false
          })
          .catch(function (error) {
            if (error.response.status === 404) {
              self.tableData = []
              self.page.dataNum = 0
              self.tableOffice = {}
              self.tableParent = {}
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
        this.page.pageNo = val
        this.loadTableData(val, true)
      },
      // tablb 表格编辑行
      editTableRowClick (index, row) {
        this.$router.push('/Main/RoleEdit/' + row.id)
      },
      bindDataScope (index, row) {
        this.$router.push('/Main/RoleBindDataScope/' + row.id)
      },
      setFunctionResource (index, row) {
        this.$router.push('/Main/FunResourceDataScopeDefine/' + row.id)
      },
      // tablb 表格删除行
      deleteTableRowClick (index, row) {
        let self = this
        this.$confirm('确定要删除吗, 是否继续?', '提示', {
          type: 'warning'
        }).then(() => {
          this.$http.delete('/base/role/' + row.id)
            .then(function (response) {
              self.$message.success('删除成功')
              // 重新加载数据
              self.searchBtnClick()
            })
            .catch(function (error) {
              if (error.response.status === 404) {
                self.$message.error('删除失败，请刷新数据再试')
              }
            })
        })
      },
      addTableRowClick () {
        loadDataControl.add(this.$store, 'RoleAddLoadData=true')
        this.$router.push('/Main/RoleAdd')
      },
      typeFormatter (row) {
        let dict = getDictByValueSync(this, 'role_type', row.type)
        return dict ? dict.name : null
      },
      disabledFormatter (row) {
        let dict = getDictByValueSync(this, 'yes_no', row.disabled)
        return dict ? dict.name : null
      },
      dataOfficeFormatter (row) {
        let dataOfficeName = null
        if (this.tableOffice && this.tableOffice[row.dataOfficeId]) {
          dataOfficeName = this.tableOffice[row.dataOfficeId].name || null
        }
        return dataOfficeName
      },
      dataParentFormatter (row) {
        let name = null
        if (this.tableParent && this.tableParent[row.parentId]) {
          name = this.tableParent[row.parentId].name || null
        }
        return name
      }
    },
    watch: {
      'searchFormModel.tableData' () {

      }
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
