<template>
  <div id="tableWrapper" class="simple-table">
    <el-table
      :key="key"
      :ref="tableConf.ref"
      :height="tableConf.height"
      :data="tableConf.data"
      :summary-method="tableConf.getSummaries"
      :show-summary="tableConf.showSummary || false"
      :render-header="renderHeader"
      :row-class-name="rowClassName"
      :header-cell-class-name="headerCellClassName"
      :row-key="tableConf.rowKey || 'id'"
      :cell-style="tableConf.cellStyle"
      highlight-current-row
      border
      class="drag_table"
      size="medium"
      @row-click="rowClick"
      @row-dblclick="rowDblclick"
      @sort-change="handleSortChange"
      @selection-change="handleSelectionChange"
      @select="handleChange"
      @select-all="handleChange"
    >
      <el-table-column
        v-if="tableConf.showSelection"
        :selectable="selectable"
        :reserve-selection="tableConf.reserveSelection || false"
        type="selection"
        align="center"
        width="50"
      />
      <el-table-column v-if="tableConf.showIndex" align="center" type="index" label="序号" width="60">
        <template slot-scope="{ row, $index }">
          <span>{{ (innerCurrentPage - 1) * innerPageSize + $index + 1 }}</span>
        </template>
      </el-table-column>
      <slot name="auditAndActiveState" />
      <template v-for="(item, index) in tableConf.colModel">
        <slot v-if="!item.hidden" :name="item.prop" :renderHeader="renderHeader" :index="index.toString()">
          <el-table-column
            v-if="!item.hidden"
            :key="index"
            :column-key="index.toString()"
            :render-header="renderHeader"
            :prop="item.prop"
            :fixed="item.fixed"
            :label="item.label"
            :align="item.align ? item.align : 'center'"
            :sortable="item.sortable ? item.sortable : false"
            :min-width="item.width"
            :show-overflow-tooltip="true"
            :formatter="item.formatter"
            :sort-orders="['ascending', 'descending']"
          />
        </slot>
      </template>
      <slot name="operation" />
      <div slot="empty" class="no-data">
        <!-- <img src="@/assets/images/no_data.png" alt="暂无数据"> -->
      </div>
    </el-table>
    <el-pagination
      prev-text="上一页"
      next-text="下一页"
      :total="innerTotal"
      :page-count="innerTotal"
      :current-page="innerCurrentPage"
      :layout="pageLayout"
      :page-size="innerPageSize"
      :page-sizes="pagination.pageSizes"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    >
    </el-pagination>
    
  </div>
</template>

<script>
export default {
  name: 'dataTable',
  components: {  },
  props: {
    paginationShow: {
      type: Boolean,
      default: true
    },
    tableConf: {
      type: Object,
      default: () => ({
        ref: {
          type: String,
          default: ''
        },
        height: {
          type: String,
          default: '100%'
        },
        data: {
          type: Array,
          default: () => []
        },
        colModel: {
          type: Array,
          default: () => []
        },
        showSelection: {
          type: Boolean,
          default: false
        },
        showIndex: {
          type: Boolean,
          default: false
        },
        preColumnsNum: {
          type: Number,
          default: 0
        }
      })
    },
    selectable: {
      type: Function,
      default: () => () => {}
    },
    pagination: {
      type: Object,
      default: () => ({
        show: true,
        currentPage: 1,
        pageSize: 50,
        pageSizes: [10, 20, 50],
        total: 0
      })
    },
    pageLayout: {
      type: String,
      default: 'total, sizes, prev, pager, next'
    },
    currentChange: {
      type: Function,
      default: () => () => {}
    },
    rowClick: {
      type: Function,
      default: () => () => {}
    },
    rowDblclick: {
      type: Function,
      default: () => () => {}
    },
    rowClassName: {
      type: Function,
      default: () => () => {}
    },
    headerCellClassName: {
      type: Function,
      default: () => () => {}
    },
    total: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      dragState: {
        start: -9, // 起始元素的 index
        end: -9, // 移动鼠标时所覆盖的元素 index
        dragging: false, // 是否正在拖动
        direction: undefined // 拖动方向
      },
      virtualStyleObj: {
        left: 'auto',
        right: 'auto',
        top: 'auto',
        height: '0px'
      },
      key: 1,
      originVirtualStyleObj: null,
      innerTotal: 0,
      innerCurrentPage: 1,
      innerPageSize: 20
    }
  },
  computed: {
    getIndex() {
      return this.tableConf.showSelection ? (this.tableConf.showIndex ? 2 : 1) : this.tableConf.showIndex ? 1 : 0
    },
    // 计算table真正的数据之前有多少列
    colModelPreColumnsNum() {
      if (this.tableConf.showSelection) {
        return this.tableConf.preColumnsNum + 2
      } else {
        return this.tableConf.preColumnsNum + 1
      }
    },
    queryInfo() {
      return {
        page: this.innerCurrentPage,
        pageSize: this.innerPageSize
      }
    }
  },
  watch: {
    'tableConf.colModel': {
      handler() {
        setTimeout(() => {
          this.$refs[this.tableConf.ref].doLayout()
        }, 0)
      },
      deep: true
    },
    total(val) {
      this.innerTotal = val
    },
    innerCurrentPage() {
      this.queryChange('page')
    },
    innerPageSize(val) {
      this.$emit('update:pageSize', val)
    }
  },
  created() {
    this.innerPageSize = this.pagination.pageSize
    // fix https://github.com/njleonzhang/vue-data-tables/issues/172
    // let totalPage = this.total / this.pagination.pageSize
    // let ceilTotalPage = Math.ceil(totalPage)

    // this.innerTotal = ceilTotalPage >= this.pagination.currentPage ? this.total : this.pagination.pageSize * this.pagination.currentPage
  },
  mounted() {
    // this.originVirtualStyleObj = Object.assign({}, this.virtualStyleObj)
    //this.queryChange('init')
  },
  methods: {
    init() {
      this.innerCurrentPage = 1
      this.queryChange('init')
    },
    queryChange(type) {
      let info = {
        type,
        ...this.queryInfo
      }
      this.$emit('query-change', info)
    },
    handleSizeChange(val) {
      this.innerPageSize = val
      this.queryChange('size')
      this.$emit('handle-size-change', val)
    },
    handleCurrentChange(val) {
      this.innerCurrentPage = val
      this.queryChange('current')
      this.$emit('handle-current-change', val)
    },
    handleFirstPage(val) {
      this.$emit('handle-first-page', val)
    },
    handleLastPage(val) {
      this.$emit('handle-last-page', val)
    },
    handleSelectionChange(val) {
      this.$emit('handle-selection-change', val)
    },
    handleSortChange({ column, prop, order }) {
      this.$emit('sort-change', { column, prop, order })
    },
    handleChange(selection, row, e) {
      this.$emit('select', selection, row, e)
      this.$emit('select-all', selection, row, e)
    },
    clearSelection() {
      this.$refs[this.tableConf.ref].clearSelection()
    },
    renderHeader(createElement, obj) {
      let label = obj.column.label
      return createElement(
        'div',

        {
          class: ['thead-cell', 'drop', 'datatable-drop'],
          domProps: {
            draggable: true,
            effectAllowed: 'all'
          },
          on: {
            // mousedown: $event => {
            //   console.log($event);
            // },
            // mouseup: $event => {},
            dragstart: $event => {
              // console.log('mousedown', obj);
              this.handleMouseDown($event, obj)
            },
            dragover: $event => {
              // console.log('dragover', obj);
              $event.preventDefault()
              this.handleMouseMove($event, obj)
            }
          }
        },
        [
          // 添加 <a> 用于显示表头 label
          label
        ]
      )
    },
    // //////////////////////////////
    // 按下鼠标开始拖动
    handleMouseDown(e, { column }) {
      this.dragState.dragging = true
      this.dragState.start = parseInt(column.columnKey)
      // 给拖动时的虚拟容器添加宽高
      const table = document.getElementsByClassName('w-table')[0]
      const virtual = document.getElementsByClassName('virtual')
      for (const item of virtual) {
        item.style.height = table.clientHeight - 1 + 'px'
        item.style.width = item.parentElement.parentElement.clientWidth + 'px'
      }
      document.addEventListener('dragend', this.handleMouseUp)
    },

    // 鼠标放开结束拖动
    handleMouseUp() {
      // document.dispatchEvent('mouseup')
      this.dragColumn(this.dragState)
      // 初始化拖动状态
      this.dragState = {
        start: -6,
        end: -6,
        dragging: false,
        direction: undefined
      }
      document.removeEventListener('dragend', this.handleMouseUp)
    },

    // 拖动中
    handleMouseMove(e, { column }) {
      if (this.dragState.dragging) {
        const index = parseInt(column.columnKey) // 记录起始列
        if (index - this.dragState.start !== 0) {
          this.dragState.direction = index - this.dragState.start < 0 ? 'left' : 'right' // 判断拖动方向
          this.dragState.end = parseInt(column.columnKey)
        } else {
          this.dragState.direction = undefined
        }
      } else {
        return false
      }
    },

    // 拖动易位
    dragColumn({ start, end }) {
      if (end < 0) return
      // const tempData = []
      // const left = direction === 'left'
      // const min = left ? end : start - 1
      // const max = left ? start + 1 : end

      var old = this.tableConf.colModel.splice(start, 1)
      this.tableConf.colModel.splice(end, 0, ...old)
      setTimeout(() => {
        // this.tableConf.colModel = JSON.parse(JSON.stringify(this.tableConf.colModel))
        // this.$forceUpdate()
        this.key = +new Date()
        window.localStorage.setItem(this.tableConf.tableId, JSON.stringify(this.tableConf.colModel))
      }, 500)
      return
    }
  }
}
</script>

<style>
</style>
