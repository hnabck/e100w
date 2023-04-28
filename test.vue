<template>
  <div>
    <a-card :bordered="false" style="margin-bottom: 10px;">
      <!-- 条件搜索 -->
      <div class="table-page-search-wrapper">
        <a-form :labelCol="labelCol" :wrapperCol="wrapperCol" ref="queryForm">
          <a-row :gutter="32">
            <a-col :span="6" >
              <a-form-item label="产品名称">
                <a-input v-model="queryParam.productName" placeholder="请输入产品名称" allow-clear @keyup.enter.native="handleQuery"/>
              </a-form-item>
            </a-col>
            <a-col :span="6" >
              <a-form-item label="排序">
                <a-input-number v-model="queryParam.treeSort" :min="0" style="width: 100%"/>
              </a-form-item>
            </a-col>
            <a-col :span="6" >
              <a-form-item label="产品状态" prop="status">
                <a-select placeholder="请选择产品状态" v-model="queryParam.status" style="width: 100%" allow-clear>
                  <a-select-option v-for="(d, index) in statusOptions" :key="index" :value="d.dictValue">{{ d.dictLabel }}</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col>
              <span class="table-page-search-submitButtons" style="float: right;">
                <a-button type="primary" @click="handleQuery"><a-icon type="search" />查询</a-button>
                <a-button style="margin-left: 8px" @click="resetQuery"><a-icon type="redo" />重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </a-card>
    <a-card :bordered="false" class="table-card">
      <!-- 增加 -->
      <sys-product-add-form
        v-if="showAddModal"
        ref="sysProductAddForm"
        :sysProductOptions="sysProductOptions"
        :statusOptions="statusOptions"
        @ok="getList"
        @select-tree="getTreeSelect"
        @close="showAddModal = false"
      />
      <!-- 编辑 -->
      <sys-product-edit-form
        v-if="showEditModal"
        ref="sysProductEditForm"
        :sysProductOptions="sysProductOptions"
        :statusOptions="statusOptions"
        @ok="getList"
        @select-tree="getTreeSelect"
        @close="showEditModal = false"
      />
      <advance-table
        :loading="loading"
        title="产品表"
        rowKey="id"
        @refresh="getList"
        :expandIconColumnIndex="1"
        :columns="columns"
        :data-source="sysProductList"
        :pagination="false"
        size="middle"
        tableKey="base-sysProduct-index-table_grid"
        :defaultExpandedRowKeys="expandedRowKeys"
        :expandedRowKeys="expandedRowKeys"
        :expandIcon="expandIcon"
        @expand="expandNode"
        :indentSize="16"
        :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }">
        <div class="table-operations" slot="button">
          <a-button type="primary" @click="handleAdd" v-hasPermi="['customer:sysProduct:add']">
            <a-icon type="plus" />新增
          </a-button>
          <a-button type="danger" v-if="!multiple" :disabled="multiple" @click="handleDelete" v-hasPermi="['customer:sysProduct:remove']">
            <a-icon type="delete" />删除
          </a-button>
          <a-button type="" @click="handleExpand" v-hasPermi="['system:testTree:query']">
            <a-icon type="caret-down" />展开
          </a-button>
          <a-button type="" @click="handleCollapse" v-hasPermi="['system:testTree:query']">
            <a-icon type="caret-up" />折叠
          </a-button>
        </div>
        <span slot="status" slot-scope="{record}">
          <a-badge :status="record.status == '0' ? 'processing' : 'error'" :text=" statusFormat(record) " />
        </span>
        <span slot="operation" slot-scope="{text, record}">
          <a @click="handleUpdate(record)" v-hasPermi="['customer:sysProduct:edit']">
            修改
          </a>
          <a-divider type="vertical" v-hasPermi="['customer:sysProduct:add']" />
          <a @click="handleAdd(record)" v-hasPermi="['customer:sysProduct:add']">
            添加子节点
          </a>
          <a-divider type="vertical" v-hasPermi="['customer:sysProduct:remove']"/>
          <a @click="handleDelete(record)" v-hasPermi="['customer:sysProduct:remove']">
            删除
          </a>
        </span>
      </advance-table>
    </a-card>
  </div>
</template>
<script>
import { listSysProduct, delSysProduct, searchSysProductList, getInitData, listTree, listTreeExcludeChild } from '@/api/customer/sysProduct'
import AdvanceTable from '@/components/pt/table/AdvanceTable'
import SysProductAddForm from '@/views/customer/sysproduct/modules/SysProductAddForm'
import SysProductEditForm from '@/views/customer/sysproduct/modules/SysProductEditForm'
import allIcon from '@/core/icons'
export default {
  name: 'SysProduct',
  components: {
    AdvanceTable,
    SysProductAddForm,
    SysProductEditForm

  },
  data () {
    return {
      showAddModal: false,
      showEditModal: false,
      allIcon,
      iconVisible: false,
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      multiple: true,
      // 选中的主键集合
      selectedRowKeys: [],
      // 选中的数据集合
      selectedRows: [],
      // 展开的主键集合
      expandedRowKeys: [],
      // label的百分比
      labelCol: { span: 6 },
      // 内容区域的百分比
      wrapperCol: { span: 18 },
      // 高级搜索 展开/关闭
      advanced: false,
      sysProductOptions: [],
      // 产品表表格数据
      sysProductList: [],
      // 产品状态字典
      statusOptions: [],
      // 查询参数
      queryParam: {
        productName: undefined,

        treeSort: undefined,

        status: undefined

      },
      columns: [
        {
          title: '产品名称',
          dataIndex: 'productName',
          ellipsis: true,
          align: 'left',
          width: '25%'
        },
        {
          title: '排序',
          dataIndex: 'treeSort',
          align: 'right',
          width: '25%'
        },
        {
          title: '产品状态',
          dataIndex: 'status',
          scopedSlots: { customRender: 'status' },
          align: 'center',
          width: '25%'
        },
        {
          title: '操作',
          dataIndex: 'operation',
          align: 'center',
          width: '25%',
          scopedSlots: { customRender: 'operation' }
        }
      ]
    }
  },
  created () {
    this.getList()
    getInitData('is_active').then(response => {
      this.statusOptions = response.data.is_active
    })
  },
  methods: {
    // 展开收缩时需要动态修改展开行集合
    expandNode (expanded, record) {
      if (expanded) {
        this.expandedRowKeys.push(record.id)
      } else {
        this.expandedRowKeys = this.expandedRowKeys.filter(
          function (item) { return item !== record.id }
        )
      }
      if (expanded && record.children.length === 0) {
        this.loading = true
        listSysProduct(this.queryParam, record.id, 1).then(response => {
          record.children = response.data
          this.loading = false
        })
      }
    },
    onSelectChange (selectedRowKeys, selectedRows, expandedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
      this.ids = this.selectedRows.map(item => item.id)
    },
    /** 查询产品表列表 */
    getList () {
      this.expandedRowKeys = []
      this.loading = true
      listSysProduct(this.queryParam).then(response => {
        console.log(response)
        this.expandTree(response.data, 1, this.expandedRowKeys)
        this.sysProductList = response.data
        this.loading = false
      })
    },
    // 产品状态字典翻译
    statusFormat (row) {
      if (row.status) {
        return this.selectDictLabel(this.statusOptions, row.status)
      } else {
        return ''
      }
    },
    /** 搜索按钮操作 */
    handleQuery () {
        if(
        (this.queryParam.productName === undefined || this.queryParam.productName === '') &&

        (this.queryParam.treeSort === undefined || this.queryParam.treeSort === '') &&

        (this.queryParam.status === undefined || this.queryParam.status === '')) {
        this.expandedRowKeys = []
        this.getList()
      } else {
        this.loading = true
        searchSysProductList(this.queryParam).then(response => {
            this.expandedRowKeys = []
            if (response.data.length !== 0) {
              this.getAllSysProductNode(response.data)
              this.sysProductList = response.data
            } else {
              this.sysProductList = []
            }
            this.loading = false
          }
        )
      }
    },
    /** 重置按钮操作 */
    resetQuery () {
      this.queryParam = {
        pageNum: 1,
        pageSize: 10,
        productName: undefined,

        treeSort: undefined,

        status: undefined

      }
      this.handleQuery()
    },
    getAllSysProductNode (nodes) {
      if (!nodes || nodes.length === 0) {
        return []
      }
      nodes.forEach(node => {
        if (node.children.length !== 0) {
          this.expandedRowKeys.push(node.id)
        }
        return this.getAllSysProductNode(node.children)
      })
    },
    toggleAdvanced () {
      this.advanced = !this.advanced
    },
    handleAdd (record) {
      this.showAddModal = true
      this.$nextTick(() => (
        this.$refs.sysProductAddForm.handleAdd(record)
      ))
    },
    handleExpand (record) {
      this.expandedRowKeys = []
      this.loading = true
      searchSysProductList(this.queryParam).then(response => {
        this.expandedRowKeys = []
        if (response.data.length !== 0) {
          this.getAllSysProductNode(response.data)
          this.sysProductList = response.data
        } else {
          this.sysProductList = []
        }
        this.loading = false
      })
    },
    handleCollapse (record) {
      this.expandedRowKeys = []
      this.loading = true
      listSysProduct(this.queryParam).then(response => {
        this.expandTree(response.data, 1, this.expandedRowKeys)
        this.sysProductList = response.data
        this.loading = false
      })
    },
    handleUpdate (record, ids) {
      this.showEditModal = true
      this.$nextTick(() => (
        this.$refs.sysProductEditForm.handleUpdate(record, ids)
      ))
    },
    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
      this.ids = this.selectedRows.map(item => item.id)
      this.multiple = !selectedRowKeys.length
    },
    /** 查询菜单下拉树结构 */
    getTreeSelect (row) {
      if (row) {
        listTreeExcludeChild(row.id, '99').then(response => {
          this.sysProductOptions = []
          const sysProduct = { id: 0, productName: '根节点', children: [] }
          sysProduct.children = response.data
          this.sysProductOptions.push(sysProduct)
        })
      } else {
        listTree('0', '99').then(response => {
          this.sysProductOptions = []
          const sysProduct = { id: 0, productName: '根节点', children: [] }
          sysProduct.children = response.data
          this.sysProductOptions.push(sysProduct)
        })
      }
    },
    /** 删除按钮操作 */
    handleDelete (row) {
      var that = this
      const sysProductIds = row.id || this.ids
      this.$confirm({
        title: '确认删除所选中数据?',
        content: '当前选中的数据',
        onOk () {
          return delSysProduct(sysProductIds)
            .then(() => {
              if (row !== null) {
                that.removeTreeNode(that.sysProductList, row)
              } else {
                 that.onSelectChange([], [], [])
                 that.getList()
              }
              that.$message.success(
                '删除成功',
                3
              )
          })
        },
        onCancel () {}
      })
    },
    expandIcon (props) {
      if (props.record.treeLeaf === 'y') {
        return <span style="margin-right:22px"></span>
      } else {
        if (props.expanded) {
          return (
            <a style="color: 'black',margin-right:0px"
               onClick={(e) => {
                 props.onExpand(props.record, e)
               }}
            >
              <a-icon type="caret-down" />
            </a>
          )
        } else {
          return (
            <a style="color: 'black' ,margin-right:0px"
               onClick={(e) => {
                 props.onExpand(props.record, e)
               }}
            >
              <a-icon type="caret-right" />
            </a>
          )
        }
      }
    }
  }
}
</script>
