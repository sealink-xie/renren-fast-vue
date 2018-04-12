

<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.name" placeholder="名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('generator:bsarea:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('generator:bsarea:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-container style="height: 500px; border: 1px solid #eee">
      <el-aside width="250px" style="background-color: rgb(238, 241, 246)">
        <el-tree
          :props="areaListTreeProps"
          node-key="areaCd"
          @node-click="handleNodeClick"
          :default-expanded-keys="[0]"
          lazy
          :load="loadNode"
          ref="tree"
          accordion >
        </el-tree>
      </el-aside>

      <el-container>
        <el-table
          :data="dataList"
          height="contentViewStyles()"
          @cell-mouse-enter="handleMouseEnter"
          border
          v-loading="dataListLoading"
          @selection-change="selectionChangeHandle"
          style="width: 100%;">
          <el-table-column
            type="selection"
            header-align="center"
            align="center"
            width="50">
          </el-table-column>

          <el-table-column
            prop="id"
            header-align="center"
            align="center"
            label="ID">
            <template slot-scope="scope">
              <el-popover
                ref="popoverLine"
                placement="right"
                width="500"
                transition="fade-in-linear"
                visible-arrow="true"
                trigger="hover">
                <el-table :data="line">
                  <el-table-column width="150" property="name" label="名称"></el-table-column>
                  <el-table-column width="150" property="areaCd" label="地区编码"></el-table-column>
                  <el-table-column width="75" property="parentId" label="上级ID"></el-table-column>
                  <el-table-column width="75" property="levels" label="层级"></el-table-column>
                </el-table>
                <el-pagination
                  @size-change="sizeChangeHandleOver"
                  @current-change="currentChangeHandleOver"
                  :current-page="pageIndexOver"
                  :page-sizes="[10]"
                  :page-size="pageSizeOver"
                  :total="totalPageOver"
                  layout="total, sizes, prev, pager, next, jumper">
                </el-pagination>
              </el-popover>
              <el-button v-popover:popoverLine type="text" size="small" >{{scope.row.id}}</el-button>
            </template>
          </el-table-column>
          <el-table-column
            prop="name"
            header-align="center"
            align="center"
            label="名称">
          </el-table-column>
          <el-table-column
            prop="areaCd"
            header-align="center"
            align="center"
            label="地区编码">
          </el-table-column>
          <el-table-column
            prop="parentId"
            header-align="center"
            align="center"
            label="上级ID">
          </el-table-column>
          <el-table-column
            prop="levels"
            header-align="center"
            align="center"
            label="层级">
          </el-table-column>
          <el-table-column
            fixed="right"
            header-align="center"
            align="center"
            width="150"
            label="操作">
            <template slot-scope="scope">
              <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
              <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>

      </el-container>
    </el-container>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="refresh"></add-or-update>
  </div>
</template>
<style>
/*.el-popover{position:absolute;background:#fff;height:600px;}*/
</style>
<script src="https://unpkg.com/element-ui/lib/index.js"></script>
<script>
  import API from '@/api'
  import AddOrUpdate from './add-or-update'
  import { treeDataTranslate } from '@/utils'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        line: [], // 行数据，悬浮使用
        areaListTreeProps: {
          label: 'name',
          children: 'children',
          isLeaf: 'leaf'
        },
        parentId: '', // 新增传递给明细的上级ID
        currentId: '', // 悬浮上级ID
        levels: '', // 新增传递给明细的层级
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        pageIndexOver: 1, // 悬浮
        pageSizeOver: 10, // 悬浮
        totalPageOver: 0, // 悬浮
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 动态加载tree数据
      loadNode (node, resolve) {
        if (node.level === 0) {
          API.bsarea.tree(999999).then(({data}) => {
            return resolve(treeDataTranslate(data.areaList, 'areaCd'))
          })
        } else {
          setTimeout(() => {
            API.bsarea.tree(parseInt(node.key)).then(({data}) => {
              return resolve(treeDataTranslate(data.areaList, 'areaCd'))
            })
          }, 100)
        }
      },
      handleNodeClick (data) {
        this.parentId = data.areaCd
        this.levels = data.levels
        this.getDataList()
      },
      // 计算列表div宽度
      contentViewStyles () {
        var height = this.$store.state.documentClientHeight
        height -= 50 // site-topbar
        height -= 40 // el-tabs__header
        height -= 15 // el-tabs__header margin-bottom
        height -= 15 // el-tabs__content padding-bottom
        height -= 2  // el-card border-top border-bottom
        height -= 100  // el-card search
        height += 'px'
        return [
          { minHeight: height }
        ]
      },
      // 更新当前节点下的数据
      refresh () {
        this.getDataList()
        API.bsarea.tree(parseInt(this.parentId)).then(({data}) => {
          this.$refs.tree.updateKeyChildren(this.parentId, treeDataTranslate(data.areaList, 'areaCd'))
        })
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        var params = {
          page: this.pageIndex,
          limit: this.pageSize,
          name: this.dataForm.name,
          parent_Id: this.parentId
        }
        API.bsarea.list(params).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 获取数据列表
      getDataListOver () {
//        this.dataListLoading = true
        var params = {
          page: this.pageIndexOver,
          limit: this.pageSizeOver,
          name: this.dataForm.name,
          parent_Id: this.currentId
        }
        API.bsarea.list(params).then(({data}) => {
          if (data && data.code === 0) {
            this.line = data.page.list
            this.totalPageOver = data.page.totalCount
          } else {
            this.line = []
            this.totalPageOver = 0
          }
//          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 每页数 悬浮
      sizeChangeHandleOver (val) {
        this.pageSizeOver = val
        this.pageIndexOver = 1
        this.getDataListOver()
      },
      // 当前页 悬浮
      currentChangeHandleOver (val) {
        this.pageIndexOver = val
        this.getDataListOver()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 悬浮数据赋值
      showLine (line) {
        this.line = line
      },
      // 获取悬浮下级数据
      showNextLevel (id) {
        this.currentId = id
        this.getDataListOver(id)
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id, this.parentId, this.levels)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          API.bsarea.del(ids).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                  this.refresh()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      handleMouseEnter (row, column, cell, event) {
        if (column.label === 'ID' && row.areaCd !== this.currentId) {
          this.showNextLevel(row.areaCd)
        }
      }
    }
  }
</script>
