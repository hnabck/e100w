<template>
  <div>
    <Panel fit="true" :bordered="false" :collapsible="true">
      <template slot="header">
        <div class="f-full" style="padding-left: 10px">
          <!--表格左边搜索框及按钮-->
          <div class="f-full" style="line-height:30px;padding: 0 10px 0 10px;float:left;">
            <a-form :form="searchForm">
              <span style="padding: 0px 5px 0px 5px" v-if="item.type==='input'" v-for="(item,k) in easyUITable.searchList" :key="k">{{ item.label }}：
                <a-input v-model="searchForm[item.prop]" :style="{width:item.width+'px'}" :placeholder="'输入'+item.label"> </a-input>
              </span>
              <span style="padding: 0px 5px 0px 5px" v-else-if="item.type==='select'">{{ item.label }}：
                <div class="f-full" style="display: inline-block">
                  <a-select default-value="lucy" v-model="searchForm[item.prop]" :style="{width:item.width+'px'}">
                    <a-select-option value="jack">
                      Jack
                    </a-select-option>
                    <a-select-option value="lucy">
                      Lucy
                    </a-select-option>
                    <a-select-option value="Yiminghe">
                      yiminghe
                    </a-select-option>
                  </a-select>
                </div>
              </span>
              <span style="padding: 0px 5px 0px 5px" v-else-if="item.type==='Date'">{{ item.label }}：
                <div class="f-full" style="display: inline-block">
                  <a-date-picker v-model="searchForm[item.prop]" :style="{width:item.width+'px'}" />
                </div>
              </span>
              <span style="padding: 0px 5px 0px 5px" v-else-if="item.type==='DateRang'">{{ item.label }}：
                <div class="f-full" style="display: inline-block">
                  <a-range-picker v-model="searchForm[item.prop]" :style="{width:item.width+'px'}" />
                </div>
              </span>
              <a-button
                style="margin-left: 2px"
                v-else-if="item.type==='button'"
                type="primary"
                @click="item.handler">
                <a-icon :type="item.icon"/>{{ item.label }}</a-button>
            </a-form>
          </div>
          <!--表格右边按钮-->
          <div style="line-height:30px;padding: 0 10px 0 10px; float:right;">
            <a-button
              style="margin-left: 2px"
              :type="bt.type"
              @click="bt.handler"
              v-for="(bt,k) in easyUITable.rightButton.data.slice(0,2)"
              :key="k"
              v-if="bt.code !=='delete'"
              v-hasPermi="[bt.hasPermi]">
              <a-icon :type="bt.icon"/>{{ bt.name }}</a-button>
            <a-button
              style="margin-left: 2px"
              :type="bt.type"
              @click="bt.handler"
              v-else-if="bt.code ==='delete'"
              :disabled="!easyUITable.multiple"
              v-hasPermi="[bt.hasPermi]">
              <a-icon :type="bt.icon"/>{{ bt.name }}</a-button>
            <a-dropdown v-if="easyUITable.rightButton.data.length>easyUITable.rightButton.showNum">
              <a-button type="default" :ghost="true" style="margin: 0px 15px 0px 2px; ">更多功能
                <a-icon type="down"/>
              </a-button>
              <a-menu slot="overlay">
                <a-menu-item v-hasPermi="[bt1.hasPermi]" v-for="(bt1,k1) in easyUITable.rightButton.data" :key="k1" v-if="k1>easyUITable.rightButton.showNum">
                  <a type="">
                    <a-icon :type="bt1.icon"/> {{ bt1.name }} </a>
                </a-menu-item>
              </a-menu>
            </a-dropdown>
          </div>
        </div>
      </template>
      <TreeGrid
        ref="tree-grid"
        :bordered="false"
        :style="{ height: `calc(100vh - `+easyUITable.tableHeight+`px)`}"
        :data="easyUITable.data"
        :idField="easyUITable.id"
        :checkbox="true"
        :cascadeCheck="false"
        :treeField="easyUITable.treeField"
        :row-class-name="getRowClass"
        @rowCheck="rowCheck($event)"
        @collapseAll="expandGrid($event)"
        @nodeContextMenu.prevent="$refs.m1.showContextMenu($event.pageX,$event.pageY)"
      >
        <GridColumn align="center" cellCss="datagrid-td-rownumber" width="30">
          <template slot="body" slot-scope="scope">
            {{ scope.row_index }}
          </template>
        </GridColumn>
        <GridColumn
          :columnResizing="false"
          :columnMoving="false"
          resizeHandle="left"
          resizable="true"
          :field="col.field"
          :align="col.align"
          v-if="k===0"
          :title="col.title"
          v-for="(col,k) in easyUITable.columns"
          :key="k">
          <template slot="header" slot-scope="scope">

            <span style="float: left;padding-left: 0px">
              <a-button-group>
                <a-button style="padding-inline-start: initial;margin-left: -4px" type="dashed" icon="caret-down" :ghost="true" @click="handleExpandAll" >
                  展开
                </a-button>
                <a-button style="padding-inline-start: initial;" icon="caret-up" type="dashed" :ghost="true" @click="handleCollapseAll" >
                  折叠
                </a-button>
              </a-button-group>
            </span>
            <span style="margin-left: 35px; line-height:32px;">{{ col.title }}</span>

          </template>
        </GridColumn>
        <GridColumn
          :field="col.field"
          :align="col.align"
          v-else
          :title="col.title"
          :key="k">
        </gridcolumn>
        <GridColumn field="cat" title="操作" align="center" width="150">
          <template slot="body" slot-scope="scope">
            <ButtonGroup style="height:24px">
              <LinkButton @click="editRow(scope.row)" >编辑</LinkButton>
              <LinkButton @click="deleteRow(scope.row)">删除</LinkButton>
            </ButtonGroup>
          </template>
        </GridColumn>
      </TreeGrid>
    </Panel>
    <!--头部面板右键菜单 菜单隐藏或显示-->
    <Menu ref="m1" :visible.sync="menuVisible" :style="menuStyle" @itemClick="onItemClick($event)">
      <MenuItem text="选择显示或隐藏字段" :disabled="true"></MenuItem>
    </Menu>
  </div>
</template>

<script>

export default {
  name: 'CustomTreeGrid',
  props:{
    tableData:{
      type:Object,
      default:null
    }
  },
  data() {
    return {
      size: 'small',
      menuVisible: false,
      menuStyle: {
        left: 0,
        top: 0,
      },
      searchForm:{},
      easyUITable:this.tableData,
      data: this.getData()
    }
  },
  methods: {
    getRowClass(row) {
      if (row.$level === 0) {
        return 'treegrid-level-0'
      } else if (row.$level === 1) {
        return 'treegrid-level-1'
      } else {
        return 'treegrid-level-2'
      }
    },

    handleCollapseAll() {
      this.$refs.tree-grid.collapseAll()
    },
    handleExpandAll() {
      this.$refs.tree-grid.expandAll()
    },
    ss() {
      const that = this
      const rowArr = that.$refs.ttt.getCheckedRows()
      console.log(rowArr)
    },
     expandGrid() {
       const nodes = this.$refs.treegrid1.ttt('getCheckedNodes')
       nodes.forEach(node => {
         node.collapse(true)
       })
    },
    checkRow(event){
      console.log(event)
    },
    // handleAdd(){
    //   console.log('handleAdd')
    // },
    getData() {
      return [
        {
          id: 1,
          name: 'C',
          size: '',
          date: '02/19/2010',
          iconCls: 'none',
          children: [
            {
              id: 2,
              name: 'Program Files',
              size: '120 MB',
              date: '03/20/2010',
              children: [
                {
                  id: 21,
                  name: 'Java',
                  size: '',
                  date: '01/13/2010',
                  state: 'closed',
                  children: [
                    {
                      id: 211,
                      name: 'java.exe',
                      size: '142 KB',
                      date: '01/13/2010'
                    },
                    {
                      id: 212,
                      name: 'jawt.dll',
                      size: '5 KB',
                      date: '01/13/2010'
                    }
                  ]
                },
                {
                  id: 22,
                  name: 'MySQL',
                  size: '',
                  date: '01/13/2010',
                  state: 'closed',
                  children: [
                    {
                      id: 221,
                      name: 'my.ini',
                      size: '10 KB',
                      date: '02/26/2009'
                    },
                    {
                      id: 222,
                      name: 'my-huge.ini',
                      size: '5 KB',
                      date: '02/26/2009'
                    },
                    {
                      id: 223,
                      name: 'my-large.ini',
                      size: '5 KB',
                      date: '02/26/2009'
                    }
                  ]
                }
              ]
            },
            {
              id: 3,
              name: 'eclipse',
              size: '',
              date: '01/20/2010',
              children: [
                {
                  id: 31,
                  name: 'eclipse.exe',
                  size: '56 KB',
                  date: '05/19/2009'
                },
                {
                  id: 32,
                  name: 'eclipse.ini',
                  size: '1 KB',
                  date: '04/20/2010'
                },
                {
                  id: 33,
                  name: 'notice.html',
                  size: '7 KB',
                  date: '03/17/2005'
                }
              ]
            }
          ]
        }
      ]
    }
  }
}
</script>

<style scoped>
.parent-component .ant-form {
  margin-bottom: 0;
}
</style>
