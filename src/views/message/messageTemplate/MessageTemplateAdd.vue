<template>
  <div class="wrapper">
    <el-form ref="form" :model="form" :rules="formRules" style="width: 460px;" label-width="100px">
      <el-form-item label="名称" prop="name" required>
        <el-input  v-model="form.name"></el-input>
      </el-form-item>
      <el-form-item label="编码" prop="code" required>
        <el-input  v-model="form.code"></el-input>
      </el-form-item>
      <el-form-item label="模板内容" prop="content" required>
        <el-input type="textarea" v-model="form.content"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addBtnClick" :loading="addLoading">添加</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import loadDataControl from '@/utils/storeLoadDataControlUtils.js'
  import SelfDictSelect from '@/components/SelfDictSelect.vue'
  export default {
    components: {
      SelfDictSelect
    },
    name: 'MessageTemplateAdd',
    data () {
      return {
        form: {
          name: null,
          code: null,
          content: null
        },
        addLoading: false,
        formRules: {
          name: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          code: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          content: [
            {required: true, message: '必填', trigger: 'blur'}
          ]
        }
      }
    },
    mounted () {
    },
    methods: {
      addBtnClick () {
        let self = this
        if (self.addLoading === false) {
          this.$refs['form'].validate((valid) => {
            if (valid) {
              // 请求添加
              self.addLoading = true
              self.$http.post('/base/message/template', self.form)
                .then(function (response) {
                  self.$message.info('消息模板添加成功')
                  self.addLoading = false
                })
                .catch(function (response) {
                  self.$message.error('消息模板添加失败，请稍后再试')
                  self.addLoading = false
                })
            } else {
              return false
            }
          })
        } else {
          self.$message.info('正在请求中，请耐心等待')
        }
      },
      resetForm () {
        this.$refs['form'].resetFields()
      }
    },
    computed: {
    },
    watch: {
    },
    beforeRouteEnter  (to, from, next) {
      next(vm => {
        // 通过 `vm` 访问组件实例
        let dataControl = 'MessageTemplateAddLoadData=true'
        if (loadDataControl.has(vm.$store, dataControl)) {
          // 重置表单
          vm.resetForm()
          loadDataControl.remove(vm.$store, dataControl)
        }
      })
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.wrapper{
  padding:1.5rem;
}
</style>
