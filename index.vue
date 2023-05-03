<template>
  <div>
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
    <custom-tree-grid
      ref="treegrid"
      :easyUITable.sync="easyUITable"
      :tableDate.sync="tableDate"
      @update:tableDate="tableDate = $event"
      :queryParam.sync="queryParam"
      :dataList.sync="sysProductList"
      @onSelectChange="onSelectChange"
    >
      <template v-slot:act="scope">
        <div class="item">
          <a-tooltip placement="topLeft">
            <template v-slot:title>
              <span>编辑</span>
            </template>
            <a @click="handleUpdate(scope.row)" v-hasPermi="['customer:sysProduct:edit']"><a-icon type="edit" /></a>
          </a-tooltip>
          <a-divider type="vertical" v-hasPermi="['customer:crmPublicCustomer:remove']"/>
          <a-tooltip placement="topLeft">
            <template v-slot:title>
              <span>删除</span>
            </template>
            <a @click="handleDelete(scope.row)" v-hasPermi="['customer:crmPublicCustomer:remove']"><a-icon type="delete" property="删除" /></a>
          </a-tooltip>
        </div>
      </template>
    </custom-tree-grid>

  </div></template>
<script>
import { listSysProduct, delSysProduct, searchSysProductList, getInitData, listTree, listTreeExcludeChild } from '@/api/customer/sysProduct'

import SysProductAddForm from '@/views/customer/sysproduct/modules/SysProductAddForm'
import SysProductEditForm from '@/views/customer/sysproduct/modules/SysProductEditForm'
import allIcon from '@/core/icons'
import CustomTreeGrid from '@/components/pt/table/CustomTreeGrid.vue'
import {traverseTreeData} from '@/utils/aidex'
export default {
  name: 'SysProduct',
  components: {
    SysProductAddForm,
    SysProductEditForm,
    CustomTreeGrid,
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
//表格参数
      tableDate:[],
      easyUITable: {
        multiple:false,
        tableHeight:'136',
        serialNumber: true,
        checkBox: true,
        border: false,
        columnResizing: 'true',
        resizeHandle:'both',
        columnMoving:true,
        multiSort: false,
        total: '',
        params:[],
        treeHandle: {
          handleCollapse:this.handleCollapse,
          handleExpand:this.handleExpand,
          onSelectChange:this.onSelectChange,
          expandNode:this.expandNode
        },
        //设置表格右键菜单
        contextMenu:[
          {
            isShow:true,
            data:[
              {text:'新增',value:'add',fun:this.handleAdd,iconCls:'my-icon-add'},
              {text:'修改',value:'edit',fun:this.handleUpdate,iconCls:'my-icon-edit'},
              {text:'删除',value:'del',fun:this.handleDelete,iconCls:'my-icon-del'},
              {text:'分隔符',value:'menuSep'},
              {text:'折叠',value:'up',fun:this.handleCollapse,iconCls:'my-icon-up'},
              {text:'展开',value:'down',fun:this.handleExpand,iconCls:'my-icon-down'},
            ]
          }
        ],
        //设置表格头部搜索框
        searchList: [
          { label: '商品名称', prop: 'productName',width:'120' ,type:'input'},
          { label: '排序', prop: 'treeSort',width:'120' ,type:'input'},
          { label: '产品状态', prop: 'status',width:'120' ,type:'select',data:[]},
          { label: '查询', prop: 'search' ,type:'button',icon:'search',handler: this.handleQuery,},
          { label: '重置', prop: 'rest' ,type:'button',icon:'redo',handler: this.resetQuery,},
        ],
        //定义表格头部右边式样及权限
        rightButton: {
          // 定义后边按钮显示的个数，从0开始计算
          showNum:1,
          data:[
            { code:'add', name: '新增', handler: this.handleAdd,type:'primary',icon:'plus',ghost:false,hasPermi:'customer:crmPublicCustomer:add'},
            { code:'delete', name: '删除', handler: this.handleDelete,type:'danger',icon:'delete',ghost:false,hasPermi:'customer:crmPublicCustomer:remove'},
            { code:'import', name: '导 入', handler: this.handleImport,type:'primary',icon:'import',ghost:false,hasPermi:'customer:crmPublicCustomer:import'},
            { code:'export', name: '导 出', handler: this.handleExport,type:'primary',icon:'download',ghost:false,hasPermi:'customer:crmPublicCustomer:edit'},
            { code:'print', name: '打 印', handler: this.handlePrint,type:'primary',icon:'printer',ghost:false,hasPermi:'customer:crmPublicCustomer:list'},
            { code:'setting', name: '设 置', handler: this.handleSetting,type:'primary',icon:'setting',ghost:false,hasPermi:'customer:crmPublicCustomer:export'},
          ]
        },
        id:'id',
        treeField:'productName',
        columns: [
          {
            title: '序号',
            field: 'rowIndex',
            type:'index',
            align: 'center',
            width: '27',
            formatter: function (row, column, cellValue, index) {
              return index
            }
          },
          {
            title: '产品名称',
            field: 'productName',
            ellipsis: true,
            align: 'left',
            width: '150',
            function:[{

            }]
          },
          {
            title: '排序',
            field: 'treeSort',
            align: 'right',
            width: '50'
          },
          {
            title: '产品状态',
            field: 'status',
            align: 'center',
            width: '100'
          },
          {
            title: '产品说明',
            field: 'remark',
            align: 'center',
            width: '300'
          },
          {
            title: '操作',
            field: 'act',
            width: '80',
            type:'slot',
            slotName:'act',
            iconCls:'icon-ok'
          }
        ]
      }
    }
  },
  created () {
    this.getList()
    getInitData('is_active').then(response => {
      this.statusOptions = response.data.is_active
      this.easyUITable.searchList.forEach((item=>{
        if (item.type==='select'){
          item.data = response.data.is_active
        }
      }))
    })
  },

  computed: {

    checkedRows() {
      return this.easyUITable.data.filter(row => row.selected)
    }
  },
  methods: {

    // 展开收缩时需要动态修改展开行集合
    expandNode (expanded, record) {
      // if (expanded) {
      //   this.expandedRowKeys.push(record.id)
      // } else {
      //   this.expandedRowKeys = this.expandedRowKeys.filter(
      //     function (item) { return item !== record.id }
      //   )
      // }
      // if (expanded && record.children.length === 0) {
      //   this.loading = true
      //   listSysProduct(this.queryParam, record.id, 1).then(response => {
      //     record.children = response.data
      //     this.loading = false
      //   })
      // }
    },

    /** 查询产品表列表 */
    getList () {
      this.expandedRowKeys = []
      this.loading = true
      listSysProduct(this.queryParam).then(response => {
        console.log(response.data)
        this.expandTree(response.data, 1, this.expandedRowKeys)
        this.sysProductList = response.data

        // this.easyUITable.data = traverseTreeData(response.data,'closed')
        this.tableDate = response.data
        this.loading = false
        // console.log(this.easyUITable.data)
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
          console.log(response)
            this.expandedRowKeys = []
            if (response.data.length !== 0) {
              this.getAllSysProductNode(response.data)
              this.sysProductList = response.data
              this.tableDate = response.data
            } else {
              this.sysProductList = []
              this.tableDate = []
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
      let myData=this.tableDate
      if (record){
        myData = traverseTreeData(myData,'open',[record])
      }else {
        myData = traverseTreeData(myData,'open',null)
      }
      this.tableDate =myData
    },
    handleCollapse (record) {
      let myData=this.tableDate
      if (record){
        myData = traverseTreeData(myData,'closed',[record])
      }else {
        myData = traverseTreeData(myData,'closed',null)
      }
      this.tableDate =myData
      console.log(myData)
    },
    handleUpdate (record, ids) {
      this.showEditModal = true
      this.$nextTick(() => (
        this.$refs.sysProductEditForm.handleUpdate(record, ids)
      ))
    },
    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      // this.selectedRows = selectedRows
      this.ids = selectedRowKeys.map(item => item.id)
      console.log(this.ids)
      this.easyUITable.multiple = !this.checkedRows.length
      console.log(this.easyUITable.multiple)
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
    // expandIcon (props) {
    //   if (props.record.treeLeaf === 'y') {
    //     return <span style="margin-right:22px"></span>
    //   } else {
    //     if (props.expanded) {
    //       return (
    //         <a style="color: 'black',margin-right:0px"
    //            onClick={(e) => {
    //              props.onExpand(props.record, e)
    //            }}
    //         >
    //           <a-icon type="caret-down" />
    //         </a>
    //       )
    //     } else {
    //       return (
    //         <a style="color: 'black' ,margin-right:0px"
    //            onClick={(e) => {
    //              props.onExpand(props.record, e)
    //            }}
    //         >
    //           <a-icon type="caret-right" />
    //         </a>
    //       )
    //     }
    //   }
    // }
  },
}
</script>
<style scoped>

</style>