<template>

  <div class="fh-page-wrapper">
    <el-form ref="form" class="fh-background-white fh-padding-30" :model="form" :rules="formRules" style="width: 460px;" label-width="100px" v-loading="formDataLoading">
      <el-form-item label="开始时间" prop="startTime" >
        <el-date-picker
          v-model="form.startTime"
          type="datetime"
          value-format="yyyy-MM-dd HH:mm:ss"
          placeholder="选择时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" prop="endTime" >
        <el-date-picker
          v-model="form.endTime"
          type="datetime"
          value-format="yyyy-MM-dd HH:mm:ss"
          placeholder="选择时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="请假类型" prop="leaveType">
        <self-dict-select v-model="form.leaveType" type="oa_workattendance_leave_type"></self-dict-select>
      </el-form-item>
      <el-form-item label="请假理由" prop="reason" >
        <el-input  v-model="form.reason"></el-input>
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
  export default {
    name: 'LeaveEdit',
    components: {
      SelfDictSelect, OfficeInputSelect},
    data () {
      return {
        // 编辑的id
        id: null,
        form: {
          startTime: null,
          endTime: null,
          leaveType: '',
          reason: null,
          updateTime: null
        },
        formDataLoading: false,
        addLoading: false,
        formRules: {
          startTime: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          endTime: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          leaveType: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          reason: [
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
        self.$http.get('/oa/workattendance/leave/' + self.id)
          .then(function (response) {
            let content = response.data.data.content
            self.form.startTime = content.startTime
            self.form.endTime = content.endTime
            self.form.leaveType = content.leaveType
            self.form.reason = content.reason
            self.form.updateTime = content.updateAt
            self.formDataLoading = false
            return Promise.resolve(content)
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
              self.$http.put('/oa/workattendance/leave/' + self.id, self.form)
                .then(function (response) {
                  self.$message.success('请假单修改成功')
                  self.addLoading = false
                })
                .catch(function (response) {
                  if (response.response.status === 400) {
                    self.$message.error('请假单已生效，不能再修改')
                  }
                  if (response.response.status === 404) {
                    self.$message.error('请假单改失败，数据不存在或已被他人修改，请刷新列表后再试')
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
      }
    },
    watch: {
    },
    // tab切换如果参数不一样，重新加载数据
    beforeRouteEnter  (to, from, next) {
      next(vm => {
        // 通过 `vm` 访问组件实例
        let dataControl = 'LeaveEditLoadData=true'
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
