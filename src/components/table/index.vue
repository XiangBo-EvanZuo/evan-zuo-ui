<!--
 * @Author: your name
 * @Date: 2021-04-10 12:31:53
 * @LastEditTime: 2021-04-11 11:35:41
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /evan_you_demo_1/src/components/table/index.vue
-->

<template>
  <div class="evan-table-container">
    <Th
      :columns="columnsList"
      :isSelectAll="isSelectAll"
      @selectAllChange="selectAllChange"
    ></Th>
    <Row
      v-for="(item, index) in tableData"
      :key="index"
      :rowData="item"
      :columns="columnsList"
      :index="index"
      :isSelectAll.sync="isSelectAll"
      @clickSubRowArrow="clickSubRowArrow(item, $event)"
      @singleTableRowChangeSelect="singleTableRowChangeSelect(item, index, $event)"
      @selectAllSubTable="selectAllSubTable"
    >
      <template v-slot:expand-table>
        <div class="sub-table-container">
          <div class="block"></div>
          <el-table
          class="sub-table"
          ref='subTable'
          :data="item.children"
          :stripe="true"
          header-row-class-name="table-header-sub-table"
          @select="subTableSelectChange(index, item, $event)"
          >
            <el-table-column
              type="selection"
              width="55">
            </el-table-column>
              <el-table-column
                prop="id"
                label="id了您嘞"
              >
              </el-table-column>
          </el-table>
        </div>
      </template>
    </Row>
    </div>
</template>

<script>
function _checkSelectAll(array) {
  return array.every(item => item.select)
}

import { Table, TableColumn, Loading } from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';


import Row from './../row'
import Th from './../th'
  export default {
    components: {
      Row,
      Th,
      [Table.name]: Table,
      [TableColumn.name]: TableColumn,
    },

    data() {
      return {
        loadingInstance: null,
        isSelectAll: false,
        columnsList: [
          {
            name: '1',
            prop: 'name1'
          },
          {
            name: '2',
            prop: 'name2'
          },
          {
            name: '3',
            prop: 'name3'
          },
        ],

        tableData: [
          {
            id: 1,
            name1: '1',
            name2: 'prop_name',
            name3: '3',
            isExpand: false,
            select: false,
            children: [],
            isLoaded: false,
          },
          {
            id: 2,
            name1: '2',
            name2: 'prop_name',
            name3: '4',
            isExpand: false,
            select: false,
            children: [],
            isLoaded: false,
          },
        ],
      }
    },

    watch: {

    },

    methods: {
      async subTableSelectChange(index, item, event) {
        console.log(index, event)
        const isSubTableSelectAll = this.$refs.subTable[index].selection.length === item.children.length
        item.select = isSubTableSelectAll
        await this.$nextTick()
        this.isSelectAll = _checkSelectAll(this.tableData)

        // function _checkSelectAll(array) {
        //   return array.every(item => item.select)
        // }
      },

      async selectAllSubTable() {
        await this.$nextTick()
        this.$refs.subTable.forEach(async (item, index) => {
          await this.selectCurrentAllSubTableRow(index)
        })
      },

      async selectAllChange(value) {
        if (value) {
          this.loadingInstance = Loading.service({ target: document.querySelector('.evan-table-container') })
          const subTableArray = await this.getAllData()
          this.tableData = this.tableData.map(item => {
            const data = subTableArray.find(each => each.parentId === item.id)?.children
            return {
              ...item,
              isLoaded: true,
              select: true,
              isExpand: true,
              children: data || [],
            }
          })
        } else {
          this.tableData = this.tableData.map(item => {
            return {
              ...item,
              select: false,
              isExpand: false,
            }
          })
        }
        this.isSelectAll = value
        await this.$nextTick()
        if (this.loadingInstance) {
          this.loadingInstance.close()
          this.loadingInstance = null
        }
      },

      async getAllData() {
        return await new Promise(resolve => {
          setTimeout(() => {
            resolve([
              {
                parentId: 1,
                children: [
                  {
                    id: 1
                  }
                ]
              },
              {
                parentId: 2,
                children: [
                  {
                    id: 2
                  }
                ]
              }
            ])
          }, 1000)
        })
      },

      async singleTableRowChangeSelect(item, subTableIndex, isSingleRowSelect) {
        console.log(item, isSingleRowSelect)
        if (isSingleRowSelect) {
          // 选中
          await this.clickSubRowArrow(item, false)
          await this.$nextTick()
          await this.selectCurrentAllSubTableRow(subTableIndex)
        } else {
          // 清除
          this.clearCurrentSubTableRowAllSelect(subTableIndex)
        }
        this.isSelectAll = _checkSelectAll(this.tableData)

        // function _checkSelectAll(array) {
        //   return array.every(item => item.select)
        // }
      },

      async selectCurrentAllSubTableRow(subTableIndex) {
        this.$refs.subTable[subTableIndex].toggleAllSelection()
        await this.$nextTick()
      },

      clearCurrentSubTableRowAllSelect(subTableIndex) {
        console.log(2)
        this.$refs.subTable[subTableIndex].clearSelection()
      },

      async clickSubRowArrow(item, isExpand) {
        console.log(item, isExpand)
        if (isExpand) {
          item.isExpand = false
          return
        } else {
          if (item.isLoaded) {
            item.isExpand = true
            return
          }
          const promise = new Promise(resolve => {
            setTimeout(() => {
              const data = {id: item.id}
              resolve(data)
            }, 1000)
          })
          const data = await promise
          item.isExpand = true
          item.isLoaded = true
          item.children.push(data)
        }
      }
    },
    // render() {
    //   const rowList = this.tableData.map(item => {
    //       return <Row rowData={item} columns={this.columnsList}></Row>
    //   })
    //   return <div class="table">
    //         <Th columns={this.columnsList}></Th>
    //         {rowList}
    //   </div>
    // },
  }
</script>

<style lang="less" scoped>
.sub-table-container {
  position: relative;
  margin-left: 50px;
}

.block {
  position: absolute;
  top: 0;
  left: 0;
  height: 46px;
  width: 50px;
  background: #f7f7f7;
  z-index: 2;
}

.table {
    display: flex;
    flex-direction: column;

    .row {
        display: flex;
        border-bottom: 1px solid #EEEEEE;
    }

    .th {
        display: flex;
        border-bottom: 1px solid #EEEEEE;
        background-color: #f7f7f7;
        height: 42px;
    }
}

</style>

<style lang="less" scoped>
::v-deep .table-header-sub-table {
  th {
    background: #f7f7f7;
  }
}
</style>