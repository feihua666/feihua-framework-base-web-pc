<template>

  <div class="fh-page-wrapper">
    <el-form ref="form" class="fh-background-white fh-padding-30" :model="form" :rules="formRules" style="width: 460px;"
             label-width="100px" v-loading="formDataLoading">
      <el-form-item label="岗位名称" prop="name">
        <el-input v-model="form.name"></el-input>
      </el-form-item>
      <el-form-item label="岗位编码" prop="code">
        <el-input v-model="form.code"></el-input>
      </el-form-item>
      <el-form-item label="岗位类型" prop="type">
        <SelfDictSelect v-model="form.type" type="post_type"></SelfDictSelect>
      </el-form-item>
      <el-form-item label="岗位职务" prop="postJobId">
        <PostJobSelect v-model="form.postJobId"></PostJobSelect>
      </el-form-item>
      <el-form-item label="是否禁用" prop="disabled">
        <SelfDictSelect v-model="form.disabled" type="yes_no"></SelfDictSelect>
      </el-form-item>
      <el-form-item label="是否公共" prop="isPublic">
        <SelfDictSelect v-model="form.isPublic" type="yes_no"></SelfDictSelect>
      </el-form-item>
      <el-form-item label="父级" prop="parentId">
        <PostInputSelect ref="postinput"  v-model="form.parentId">
        </PostInputSelect>
      </el-form-item>
      <el-form-item label="归属机构" prop="dataOfficeId">
        <OfficeInputSelect ref="officeinput"  v-model="form.dataOfficeId">
        </OfficeInputSelect>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-check" @click="updateBtnClick" :loading="addLoading">修改</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import SelfDictSelect from '@/components/SelfDictSelect.vue'
  import OfficeInputSelect from '@/views/office/OfficeInputSelect'
  import PostInputSelect from '@/views/postjob/PostInputSelect.vue'
  import PostJobSelect from '@/views/postjob/PostJobSelect.vue'
  export default {
    name: 'PostEdit',
    components: {
      SelfDictSelect,
      PostInputSelect,
      PostJobSelect,
      OfficeInputSelect
    },
    data () {
      return {
        // 编辑的id
        id: null,
        form: {
          id: '',
          name: '',
          code: '',
          type: '',
          postJobId: '',
          disabled: '',
          parentId: '0',
          dataOfficeId: '',
          isPublic: '',
          updateTime: null
        },
        formDataLoading: false,
        addLoading: false,
        formRules: {
          name: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          code: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          disabled: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          dataOfficeId: [
            {required: true, message: '必填', trigger: 'blur'}
          ]
        }
      }
    },
    mounted () {
      this.id = this.$route.params.id
      this.loadEditData(this.id)
    },
    methods: {
      loadEditData (id) {
        if (this.formDataLoading === true) {
          return
        }
        this.resetForm()
        let self = this
        self.formDataLoading = true
        self.$http.get('/base/postjob/post/' + self.id)
          .then(function (response) {
            let content = response.data.data.content
            self.form.id = content.id
            self.form.name = content.name
            self.form.code = content.code
            self.form.type = content.type
            self.form.postJobId = content.postJobId
            self.form.disabled = content.disabled
            self.form.parentId = content.parentId
            self.form.dataOfficeId = content.dataOfficeId
            self.form.updateTime = content.updateAt
            self.formDataLoading = false
            // 父级回回显
            self.$refs.postinput.initLabelName(content.parentId)
            // 机构回显
            self.$refs.officeinput.initLabelName(content.dataOfficeId)
          })
          .catch(function (response) {
            self.formDataLoading = false
          })
      },
      updateBtnClick () {
        let self = this
        if (self.addLoading === false) {
          this.$refs['form'].validate((valid) => {
            if (valid) {
              // 请求添加
              self.addLoading = true
              self.$http.put('/base/postjob/post/' + self.id, self.form)
                .then(function (response) {
                  self.$message.success('岗位修改成功')
                  self.addLoading = false
                })
                .catch(function (response) {
                  if (response.response.status === 404) {
                    self.$message.error('岗位改失败，数据不存在或已被他人修改，请刷新列表后再试')
                  }
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
        this.$refs.officeinput.setLabelName(null)
        this.$refs.postinput.setLabelName(null)
      }
    },
    watch: {
    },
    // tab切换如果参数不一样，重新加载数据
    beforeRouteEnter  (to, from, next) {
      next(vm => {
        // 通过 `vm` 访问组件实例
        let dataControl = 'PostEditLoadData=true'
        if (vm.id !== vm.$route.params.id || vm.$utils.loadDataControl.has(dataControl)) {
          vm.id = vm.$route.params.id
          vm.loadEditData(vm.id)
          vm.$utils.loadDataControl.remove(dataControl)
        }
      })
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
