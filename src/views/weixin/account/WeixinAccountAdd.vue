<template>

  <div class="fh-page-wrapper">
    <el-form ref="form" class="fh-background-white fh-padding-30" :model="form" :rules="formRules" style="width: 460px;" label-width="100px">
      <el-form-item label="账号" prop="account" >
        <el-input v-model="form.account"></el-input>
      </el-form-item>
      <el-form-item label="名称" prop="name" >
        <el-input v-model="form.name"></el-input>
      </el-form-item>
      <el-form-item label="微信号" prop="weixinId">
        <el-input v-model="form.weixinId"></el-input>
      </el-form-item>
      <el-form-item label="TOKEN" prop="token" >
        <el-input v-model="form.token"></el-input>
      </el-form-item>
      <el-form-item label="APPID" prop="appid" >
        <el-input v-model="form.appid"></el-input>
      </el-form-item>
      <el-form-item label="APPSECRET" prop="appsecret" >
        <el-input v-model="form.appsecret"></el-input>
      </el-form-item>
      <el-form-item label="WHICH" prop="which" >
        <el-input v-model="form.which"></el-input>
      </el-form-item>
      <el-form-item label="类型" prop="type" >
        <self-dict-select v-model="form.type" type="weixin_account_type"/>
      </el-form-item>
      <el-form-item label="是否有效" prop="status" >
        <self-dict-select v-model="form.status" type="yes_no"/>
      </el-form-item>
      <el-form-item label="认证" prop="auth" >
        <self-dict-select v-model="form.auth" type="yes_no"/>
      </el-form-item>
      <el-form-item label="欢迎语类型" prop="templateType" v-if="typeLimit.templateType" >
        <self-dict-select v-model="form.templateType" type="weixin_msg_type"/>
      </el-form-item>
      <el-form-item label="欢迎语" prop="template" v-if="typeLimit.template" >
        <el-input type="textarea" :autosize="{ minRows: 2}" v-model="form.template"></el-input>
      </el-form-item>
      <el-form-item label="消息类型" prop="msgType" v-if="typeLimit.msgType" >
        <self-dict-select v-model="form.msgType" type="xml_or_json_type"/>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input type="textarea" :autosize="{ minRows: 2}" v-model="form.remark"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-check" @click="addBtnClick" :loading="addLoading">添加</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import SelfDictSelect from '@/components/SelfDictSelect.vue'

  export default {
    components: {SelfDictSelect},
    name: 'WeixinAccountAdd',
    data () {
      return {
        form: {
          account: null,
          name: null,
          weixinId: null,
          token: null,
          appid: null,
          appsecret: null,
          which: null,
          type: null,
          status: null,
          auth: null,
          remark: null,
          msgType: null,
          template: null,
          templateType: null
        },
        addLoading: false,
        typeLimit: {
          msgType: false,
          template: false,
          templateType: false
        },
        formRules: {
          account: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          name: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          token: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          appid: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          appsecret: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          which: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          type: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          status: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          auth: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          msgType: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          template: [
            {required: true, message: '必填', trigger: 'blur'}
          ],
          templateType: [
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
              self.$http.postJson('/weixinaccount/account', self.form)
                .then(function (response) {
                  self.$message.success('微信公众账号添加成功')
                  self.addLoading = false
                })
                .catch(function (response) {
                  self.$message.error('微信公众账号添加失败，请稍后再试')
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
    beforeRouteEnter (to, from, next) {
      next(vm => {
        // 通过 `vm` 访问组件实例
        let dataControl = 'WeixinAccountAddLoadData=true'
        if (vm.$utils.loadDataControl.has(dataControl)) {
          // 重置表单
          vm.resetForm()
          vm.$utils.loadDataControl.remove(dataControl)
        }
      })
    },
    watch: {
      'form.type' (value) {
        let self = this
        let _typeLimit = {
          msgType: false,
          template: false,
          templateType: false
        }
        switch (value) {
          case 'weixin_miniprogram':
            _typeLimit.msgType = true
            break
          case 'weixin_publicplatform':
            _typeLimit.template = true
            _typeLimit.templateType = true
            break
        }

        // 将数据清除
        for (let key in _typeLimit) {
          if (_typeLimit[key] === false) {
            self.form[key] = null
          }
        }
        self.typeLimit = _typeLimit
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
