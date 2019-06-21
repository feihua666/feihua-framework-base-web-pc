<template>
  <div>
    <el-popover
      ref="treeSelect"
      placement="right"
      trigger="click">
      <el-scrollbar wrapStyle="max-height:500px;">
        <UserGroupTree v-on:nodeClick="treeNodeClick"></UserGroupTree>
      </el-scrollbar>
    </el-popover>
    <el-input style="display: none;" value="" v-model="model">
    </el-input>
    <el-input   value="" v-model="name" v-on:focus="handleFocus($event)" v-on:handleBlur="handleBlur($event)" v-on:change="emitChange" v-on:input="emitInput" :readonly="true" clearable>
      <el-button slot="append" icon="el-icon-search" v-popover:treeSelect></el-button>
      <i slot="suffix" class="el-input__icon el-icon-circle-close el-input__clear" @click="setLabelName(null)"></i>
    </el-input>
  </div>
</template>

<script>
  import UserGroupTree from '@/views/group/UserGroupTree.vue'
  export default {
    components: {UserGroupTree},
    name: 'UserGroupInputSelect',
    props: {
      value: '',
      labelName: ''
    },
    data () {
      return {
        model: '',
        name: ''
      }
    },
    mounted () {
    },
    methods: {
      handleFocus (event) {
        this.$emit('focus', event)
      },
      handleBlur (event) {
        this.$emit('blur', event)
      },
      emitChange (val) {
        this.name = val
        this.$emit('change', this.model)
      },
      emitInput (val) {
        this.name = val
        this.$emit('input', this.model)
      },
      treeNodeClick (data) {
        this.model = data.id
        this.name = data.name
        this.$emit('change', this.model)
        this.$emit('input', this.model)
      },
      setLabelName (name) {
        this.name = name
      },
      initLabelName (id) {
        let _id = id
        if (!_id) {
          _id = this.model
        }
        if (_id) {
          this.$http.get('/base/group/' + _id).then(response => {
            let content = response.data.data.content
            this.name = content.name
          }).catch(() => {
            this.name = null
          })
        } else {
          this.name = null
        }
      }
    },
    watch: {
      value (val) {
        this.model = val
      },
      name (val) {
        if (val === '' || val == null) {
          this.model = ''
          this.$emit('change', this.model)
          this.$emit('input', this.model)
        }
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
