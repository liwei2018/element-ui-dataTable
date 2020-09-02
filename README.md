## 简介

我们平时开用vue和element-ui开发后端管理系统的时候需要维护表格属性和分页对象造成了代码的冗余，这个组件解决的问题主要是帮助开发者更专注的关系业务逻辑

### 使用方式


```js
<data-table
    :ref="tableConf.ref"
    key="dataTable"
    :table-conf="tableConf"
    @handle-selection-change="handleSelectionChange"
    :row-dblclick="rowDblclick"
    @query-change="queryList"
    :total="total"
    :pagination="tableConf.pagination"
>
</data-table>
 
...
...
data() {
    return {
      tableConf: {
        reserveSelection: true,
        tableId: 'manufacturerList',
        height: '100%',
        ref: 'manufacturerTable',
        showIndex: true,
        showSelection: true,
        currRow: {},
        data: [],
        colModel: [
          {
            label: '生产厂家编号',
            prop: 'pref',
            hidden: false,
            align: 'left',
            width: 120
          },
          {
            label: '生产厂家名称',
            prop: 'name',
            hidden: false,
            align: 'left',
            width: 120
          },
          {
            label: '联系人姓名',
            prop: 'contactName',
            hidden: false,
            align: 'left',
            width: 120
          },
          {
            label: '联系人电话',
            prop: 'contactPhone',
            hidden: false,
            align: 'left',
            width: 120
          },
          {
            label: '邮箱',
            prop: 'email',
            hidden: false,
            align: 'left',
            width: 120
          },
          {
            label: '地址',
            prop: 'detailedAddress',
            hidden: false,
            align: 'left',
            width: 120
          },
          {
            label: '创建日期',
            prop: 'createTime',
            hidden: false,
            align: 'center',
            width: 150,
            formatter: (row, column, cellValue, index) => {
              return parseTime(cellValue)
            }
          },
          {
            label: '备注',
            prop: 'remark',
            hidden: false,
            align: 'left',
            width: 150
          }
        ],
        pagination: {
          currentPage: 1,
          pageSize: 1,
          pageSizes: [1, 10, 20],
          lastPage: 1,
          total: 0
        }
      },
      total: 0
    }
  },
  methods: {
    async queryList(pageInfo) {
      if (!pageInfo) {
        this.$refs.manufacturerTable.init()
        return
      }
      const { pageSize: pageSize, page: page } = pageInfo
      const params = { ...this.formModel, rows: pageInfo.pageSize, page: pageInfo.page }
      delete params.createTime
 
      this.loading = true
      this.loadingText = '加载中...'
      try {
        const { list, total } = await queryManufactorList(params)
        this.loading = false
        //this.tableConf.pagination = { total, currentPage, lastPage, pageSize }
        this.tableConf.data = list
        this.total = total
      } catch (error) {
        this.loading = false
      }
    }
}
```
### 参数说明

| 参数 | 说明 |
| ------- | ------- |
|    queryList     |   搜索方法      |
|    total     |   总条数      |
|    tableConf.ref     |   表格ref      |
|    tableConf.height     |   表格高度      |
|    tableConf.data     |   表格数据      |
|    tableConf.colModel     |   表格列      |
|    tableConf.showSelection     |   是否显示选择框      |
|    tableConf.showIndex     |   是否显示序号      |
|    tableConf.reserveSelection     |   保留之前选中的数据      |

