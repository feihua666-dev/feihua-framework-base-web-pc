<template>
<div>
  <el-input style="width:200px;"
    placeholder="输入关键字进行过滤"
    v-model="treeFilterText" clearable>
    <el-button slot="append" icon="el-icon-refresh" @click="treeRefreshBtnClick"></el-button>
  </el-input>
  <el-tree v-loading="treeLoading" :data="treeData" :props="treeDefaultProps"
           @node-click="treeNodeClick"
           :filter-node-method="treeFilterNode"
           ref="tree"
           node-key="id"
           :expand-on-click-node="false" :default-expanded-keys="defaultExpandedKeys">

  </el-tree>
</div>
</template>

<script>
  import { arrayToTree } from '@/utils/treeUtils.js'
  export default {
    name: 'WeixinMenuTree',
    props: {
      loadData: {
        default: true
      },
      which: {
        default: null
      },
      watchWhich: {
        default: true
      }
    },
    data () {
      return {
        defaultExpandedKeys: [],
        // 左边树搜索文本
        treeFilterText: '',
        // 左边树数据
        treeData: null,
        treeLoading: false,
        // 树属性，关键字属性
        treeDefaultProps: {
          children: 'children',
          label: 'name'
        },
        param: {
          which: null,
          orderable: true,
          orderby: 'sequence-asc'
        }
      }
    },
    mounted () {
      if (this.loadData === true) {
        this.loadTreeData()
      }
    },
    methods: {
      // 左边树过滤匹配
      treeFilterNode (value, data) {
        if (!value) return true
        return data.name.indexOf(value) !== -1
      },
      // 点击树节点事件
      treeNodeClick (data) {
        this.$emit('nodeClick', data)
      },
      // 左边刷新按钮，重新加载树数据
      treeRefreshBtnClick () {
        console.log(this.which)
        this.loadTreeData()
      },
      // 加载树数据
      loadTreeData () {
        let self = this
        if (!this.which) {
          self.treeData = []
          return
        }
        self.treeLoading = true

        this.$http.get('/weixinmenu/menus', this.param)
          .then(function (response) {
            let content = response.data.data.content
            self.treeData = content
            self.treeLoading = false
          })
          .catch(function (error) {
            if (error.response.status === 404) {
              self.treeData = []
            }
            self.treeLoading = false
          })
      }
    },
    watch: {
      // 左边树过滤筛选
      treeFilterText (val) {
        this.$refs.tree.filter(val)
      },
      treeData (data) {
        // 数据转树结构
        let treeData = arrayToTree(data)
        // 默认展开不多于5个的根节点
        this.defaultExpandedKeys = []
        for (let i = 0; i < treeData.length; i++) {
          this.defaultExpandedKeys.push(treeData[i].id)
          if (this.defaultExpandedKeys.length > 5) {
            break
          }
        }
        return treeData
      },
      which (val) {
        if (this.watchWhich) {
          this.loadTreeData()
        }
        this.param.which = val
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
