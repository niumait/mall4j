<template>
  <div class="mod-transport">
    <avue-crud ref="crud"
               :page="page"
               :data="dataList"
               :option="tableOption"
               @search-change="searchChange"
               @selection-change="selectionChange"
               @on-load="getDataList">
      <template slot-scope="scope"
                slot="prodPropValues">
        <el-tag v-for="item in scope.row.prodPropValues"
                :key="item.valueId">{{item.propValue}}
        </el-tag>
      </template>
      <template slot="menuLeft">
        <el-button type="primary"
                   icon="el-icon-plus"
                   size="small"
                   v-if="isAuth('shop:transport:save')"
                   @click.stop="addOrUpdateHandle()">新增
        </el-button>

        <el-button type="danger"
                   @click="deleteHandle()"
                   v-if="isAuth('shop:transport:delete')"
                   size="small"
                   :disabled="dataListSelections.length <= 0">批量删除
        </el-button>
      </template>

      <template slot-scope="scope"
                slot="menu">
        <el-button type="primary"
                   icon="el-icon-edit"
                   size="small"
                   v-if="isAuth('shop:transport:update')"
                   @click.stop="addOrUpdateHandle(scope.row.transportId)">修改
        </el-button>

        <el-button type="danger"
                   icon="el-icon-delete"
                   size="small"
                   v-if="isAuth('shop:transport:delete')"
                   @click.stop="deleteHandle(scope.row.transportId)">删除
        </el-button>
      </template>
    </avue-crud>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible"
                   ref="addOrUpdate"
                   @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import {tableOption} from '@/crud/shop/transport'
import AddOrUpdate from './transport-add-or-update'
  export default {
    data() {
      return {
        dataForm: {
          transName: ''
        },
        dataList: [],
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        page: {
          total: 0, // 总页数
          currentPage: 1, // 当前页数
          pageSize: 10 // 每页显示多少条
        },
        tableOption: tableOption
      }
    },
    components: {
      AddOrUpdate
    },
    methods: {
      // 获取数据列表
      getDataList(page, params) {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/shop/transport/page'),
          method: 'get',
          params: this.$http.adornParams(
            Object.assign(
              {
                current: page == null ? this.page.currentPage : page.currentPage,
                size: page == null ? this.page.pageSize : page.pageSize
              },
              params
            )
          )
        }).then(({data}) => {
          this.dataList = data.records
          this.page.total = data.total
          this.dataListLoading = false
        }
      )
      },
      // 新增 / 修改
      addOrUpdateHandle(id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        }
      )
      },
      // 删除
      deleteHandle(id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {return item.transportId}
      )
        this.$confirm(`确定进行[${id ? '删除' : '批量删除'}]操作?`, '提示',
          {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }
        )
          .then(() => {
          this.$http({
            url: this.$http.adornUrl('/shop/transport'),
            method: 'delete',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose:() => {
              // this.getDataList(this.page)
              this.refreshChange()
            }
          }
      )
      })
      }).catch(() => {}
      )
      },

      // 条件查询
      searchChange(params) {
        this.getDataList(this.page, params)
      },
      // 刷新回调用
      refreshChange() {
        this.page = this.$refs.crud.$refs.tablePage.defaultPage
        this.getDataList(this.page)
        this.dataListSelections = []
        this.$refs.crud.selectClear()
      },
      // 多选变化
      selectionChange(val) {
        this.dataListSelections = val
      }
    }
  }
</script>
