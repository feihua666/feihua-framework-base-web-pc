<template>

    <el-container class="fh-page-wrapper">
      <el-main>
        <el-collapse value="1">
          <el-collapse-item title="查询条件" name="1">
            <el-form ref="searchForm" :model="searchFormModel" :inline="true" size="small">
              <el-form-item label="名称">
                <el-input  v-model="searchFormModel.name"></el-input>
              </el-form-item>
              <el-form-item label="类型">
                <self-dict-select v-model="searchFormModel.type" type="dataScope_type"></self-dict-select>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" icon="el-icon-search" :loading="tableLoading" @click="searchBtnClick">查询</el-button>
                <el-button type="primary" icon="el-icon-plus" @click="addTableRowClick">添加</el-button>
              </el-form-item>
            </el-form>
          </el-collapse-item>
        </el-collapse>
        <self-table :columns="columns" :tableData="tableData" :page="page" :table-loading="tableLoading" v-on:pageSizeChange="pageSizeChange" v-on:pageNoChange="pageNoChange"></self-table>
      </el-main>
    </el-container>

</template>

<script>
  import SelfPage from '@/components/SelfPage.vue'
  import SelfTable from '@/components/SelfTable.vue'
  import SelfDictSelect from '@/components/SelfDictSelect.vue'
  export default {
    name: 'DataScope',
    components: {
      SelfDictSelect,
      SelfTable,
      SelfPage
    },
    data () {
      return {
        columns: [
          {
            name: 'name',
            label: '名称'
          },
          {
            name: 'type',
            label: '类型',
            dict: 'dataScope_type'
          },
          {
            name: 'dataOfficeId',
            label: '机构',
            formatter: this.dataOfficeFormatter
          },
          {
            label: '操作',
            width: '300',
            buttons: [
              {
                label: '编辑',
                styleType: 'primary',
                icon: 'el-icon-edit',
                click: this.editTableRowClick
              },

              {
                label: '数据范围设置',
                styleType: 'primary',
                icon: 'el-icon-setting',
                click: this.dataScopeClick
              },
              {
                label: '删除',
                styleType: 'danger',
                icon: 'el-icon-delete',
                click: this.deleteTableRowClick
              }
            ]
          }
        ],
        page: {
          dataNum: 0
        },
        tableOffice: {},
        // 表格数据
        tableData: [],
        tableLoading: false,
        // 搜索的查询条件
        searchFormModel: {
          name: '',
          type: '',
          includeOffice: true,
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
        this.$http.get('/base/dataScopes', self.searchFormModel)
          .then(function (response) {
            let content = response.data.data.content
            self.tableOffice = response.data.data.office
            self.tableData = content
            self.page.dataNum = response.data.data.page.dataNum
            self.tableLoading = false
          })
          .catch(function (error) {
            if (error.response.status === 404) {
              self.tableData = []
              self.page.dataNum = 0
              self.tableOffice = {}
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
      // tablb 表格编辑行
      editTableRowClick (index, row) {
        this.$utils.loadDataControl.add('DataScopeEditLoadData=true')
        this.$router.push('/Main/DataScopeEdit/' + row.id)
      },
      // tablb 表格删除行
      deleteTableRowClick (index, row) {
        let self = this
        this.$confirm('确定要删除吗, 是否继续?', '提示', {
          type: 'warning'
        }).then(() => {
          self.$http.delete('/base/dataScope/' + row.id)
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
        this.$utils.loadDataControl.add('DataScopeAddLoadData=true')
        this.$router.push('/Main/DataScopeAdd')
      },
      dataScopeClick (index, row) {
        this.$router.push('/Main/DataScopeSetting/' + row.id)
      },
      dataOfficeFormatter (row) {
        let name = null
        if (this.tableOffice && this.tableOffice[row.dataOfficeId]) {
          name = this.tableOffice[row.dataOfficeId].name || null
        }
        return name
      }
    },
    watch: {
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
