<template>
  <page-header-wrapper>
    <a-card :bordered="false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="48">
            <a-col :md="8" :sm="24">
              <a-form-item label="User name">
                <a-input v-model="queryParam.name" placeholder=""/>
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="Status">
                <a-select v-model="queryParam.status" placeholder="Status" default-value="enabled">
                  <a-select-option value="enabled">Enabled</a-select-option>
                  <a-select-option value="disabled">Disabled</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="!advanced && 8 || 24" :sm="24">
              <span class="table-page-search-submitButtons" :style="advanced && { float: 'right', overflow: 'hidden' } || {} ">
                <a-button type="primary" @click="$refs.table.refresh(true)">Filter</a-button>
                <a-button style="margin-left: 8px" @click="() => this.queryParam = {}">Clean</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <div class="table-operator">
        <a-button type="primary" icon="plus" @click="handleAdd">Add</a-button>
        <a-button :disabled="selectedRowKeys.length != 1" type="primary" icon="plus"  @click="handleEdit">Edit</a-button>
      </div>

     

      <s-table
        ref="table"
        size="default"
        rowKey="key"
        :columns="columns"
        :data="loadData"
        :alert="false"
        :rowSelection="rowSelection"
        showPagination="auto"
      >        
        <span slot="status" slot-scope="text">
          <a-badge v-if="text=='disabled'" status="default" :text="text" />
          <a-badge v-else status="success" :text="text" />
        </span>
        <span slot="action" slot-scope="text, record">
          <template>
            <a @click="handleEdit(record)">Edit</a>
            <a-divider type="vertical" />
            <a v-if="record.status == 'enabled'" @click="handleEnableDisable(record)">Disable</a>
            <a v-else @click="handleEnableDisable(record)">Enable</a>                        
          </template>
        </span>
      </s-table>

      <create-form
        ref="createModal"
        :visible="visible"
        :loading="confirmLoading"
        :model="mdl"
        @cancel="handleCancel"
        @ok="handleOk"
      />
      <step-by-step-modal ref="modal" @ok="handleOk"/>
    </a-card>
  </page-header-wrapper>
</template>

<script>
import moment from 'moment'
import { STable, Ellipsis } from '@/components'
import { getRoleList, getServiceList } from '@/api/manage'

import StepByStepModal from './modules/StepByStepModal'
import CreateForm from './modules/CreateForm'

const columns = [
  {
    title: 'User Id',
    dataIndex: 'id'
  },
  {
    title: 'Name',
    dataIndex: 'name'
    // scopedSlots: { customRender: 'description' }
  }, 
  {
    title: 'Last Name',
    dataIndex: 'lastname'
  }, 
  {
    title: 'Login',
    dataIndex: 'login'
  },
  {
    title: 'Profile',
    dataIndex: 'profile'
  },
  {
    title: 'Updated At',
    dataIndex: 'updatedAt',
    sorter: true
  },
  {
    title: 'Status',
    dataIndex: 'status',
    sorter: true,
    scopedSlots: { customRender: 'status' }
  },
  {
    title: 'Action',
    dataIndex: 'action',
    width: '150px',
    scopedSlots: { customRender: 'action' }
  }
]

const statusMap = {
  0: {
    status: 'default',
    text: '关闭'
  },
  1: {
    status: 'processing',
    text: '运行中'
  },
  2: {
    status: 'success',
    text: '已上线'
  },
  3: {
    status: 'error',
    text: '异常'
  }
}

export default {
  name: 'TableList',
  components: {
    STable,
    Ellipsis,
    CreateForm,
    StepByStepModal
  },
  data () {
    this.columns = columns
    return {
      // create model
      visible: false,
      confirmLoading: false,
      mdl: null,
      // 高级搜索 展开/关闭
      advanced: false,
      // 查询参数
      queryParam: {},
      // 加载数据方法 必须为 Promise 对象
      loadData: parameter => {
        const requestParameters = Object.assign({}, parameter, this.queryParam)
        console.log('queryParam:', this.queryParam)
        console.log('loadData request parameters:', requestParameters)
        console.log('parameter:', parameter)
        return getServiceList(requestParameters)
          .then(res => {
            return res.result
          })
      },
      selectedRowKeys: [],
      selectedRows: []
    }
  },
  filters: {
    statusFilter (type) {
      return statusMap[type].text
    },
    statusTypeFilter (type) {
      return statusMap[type].status
    }
  },
  created () {
    getRoleList({ t: new Date() })
  },
  computed: {
    rowSelection () {
      return {
        selectedRowKeys: this.selectedRowKeys,
        onChange: this.onSelectChange
      }
    }
  },
  methods: {
    handleAdd () {
      this.mdl = null
      this.visible = true
    },
    handleEdit (record) {
      this.visible = true
      this.mdl = { ...record }
    },
    handleOk () {
      const form = this.$refs.createModal.form
      this.confirmLoading = true
      form.validateFields((errors, values) => {
        if (!errors) {
          console.log('values', values)          
          if (values.id > 0) {
            // 修改 e.g.
            new Promise((resolve, reject) => {
              setTimeout(() => {
                resolve()
              }, 1000)
            }).then(res => {
              this.visible = false
              this.confirmLoading = false
              // 重置表单数据
              form.resetFields()
              // 刷新表格
              this.$refs.table.refresh()

              this.$message.info('Successfully update')
            })
          } else {
            // 新增
            new Promise((resolve, reject) => {
              setTimeout(() => {
                resolve()
              }, 1000)
            }).then(res => {
              this.visible = false
              this.confirmLoading = false
              // 重置表单数据
              form.resetFields()
              // 刷新表格
              this.$refs.table.refresh()

              this.$message.info('Successfully adition')
            })
          }
        } else {
          this.confirmLoading = false
        }
      })      
    },
    handleCancel () {
      this.visible = false

      const form = this.$refs.createModal.form
      form.resetFields()
    },
    handleEnableDisable (record) {
      if (record.status === 'enabled' ) {
        record.status = 'disabled'
        this.$message.success(`${record.name} User disabled`)
      } else {
        record.status = 'enabled'
        this.$message.success(`${record.name} User enabled`)
      }
    },
    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
    toggleAdvanced () {
      this.advanced = !this.advanced
    },
    resetSearchForm () {
      this.queryParam = {
        date: moment(new Date())
      }
    }
  }
}
</script>
