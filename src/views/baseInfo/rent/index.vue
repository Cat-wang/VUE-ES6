<template>
  <div class="app-container">
    <v-pageSearch>
      <el-form :inline="true">
        <el-form-item :label="'货主：'" >
          <select-basics
            v-model="formSearch.supplierId"
            type="supplier"
            placeholder="请选择"
            style-name="width:150px;padding-right:0;"
          />
        </el-form-item>

        <el-form-item :label="'仓库：'">
          <select-basics
            v-model="formSearch.warehouseId"
            style-name="width:150px;padding-right:0;"
            type="store"
            placeholder="请选择"
            clearable
          />
        </el-form-item>
        <el-button
          type="primary"
          icon="el-icon-search"
          @click="onSearch"
        >查询</el-button>

      </el-form>
    </v-pageSearch>
    <v-pageToolbar align="left">
      <el-button type="primary" icon="el-icon-tickets" @click="onOpen">新增</el-button>
      <el-button :disabled="!table.body.length > 0" @click="onExport">导出</el-button>
    </v-pageToolbar>

    <v-pageTable>
      <el-table v-loading="table.loading" :data="table.body" :max-height="table.autoHeight" :header-cell-style="$store.state.element.headerBg" tooltip-effect="dark" style="width: 100%" @selection-change="selec" >
        <el-table-column type="selection" fixed width="55"/>
        <el-table-column prop="supplierName" min-width="80" label="货主"/>
        <el-table-column prop="warehouseName" min-width="80" label="仓库"/>
        <el-table-column prop="price" min-width="80" label="租赁单价"/>
        <el-table-column prop="createTime" min-width="80" label="创建时间">
          <template slot-scope="scope">
            {{ timeToStr(scope.row.createTime,'{y}-{m}-{d}') }}
          </template>
        </el-table-column>
        <el-table-column :label="'操作'" fixed="right" width="200">
          <template slot-scope="scope">
            <el-button :title="'编辑'" type="primary" size="mini" icon="el-icon-edit" @click="onCompile(scope.row)"/>
            <el-button :title="'删除'" type="primary" size="mini" icon="el-icon-delete" @click="onDelete(scope.row)"/>
          </template>
        </el-table-column>
      </el-table>
      <template slot="pagination" slot-scope="props">
        <el-pagination :layout="props.layout" :page-sizes="props.sizes" :current-page.sync="table.currentPage" :total="table.paginationTotal" background @current-change="pageChange" @size-change="sizeChange" />
      </template>
    </v-pageTable>

    <el-dialog id="form" :title="popupTitle" :visible.sync="popupState" :close-on-click-modal="false" :modal="false" top="10vh" width="400px" @close="onClose">
      <el-form ref="ruleForm" :model="popupData" :rules="rules" label-width="140px" label-position="right">
        <el-row>
          <el-col :span="24">
            <el-form-item label="货主：" prop="supplierId" >
              <select-basics
                v-model="popupData.supplierId"
                type="supplier"
                placeholder="请选择"
                style-name="width:180px;padding-right:0;"
              />
            </el-form-item>
          </el-col>

        </el-row>
        <el-row>
          <el-col :span="24">
            <el-form-item label="仓库：" prop="warehouseId" >
              <select-basics
                v-model="popupData.warehouseId"
                type="store"
                placeholder="请选择"
                style-name="width:180px;padding-right:0;"
              />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
            <el-form-item label="租凭单价：" prop="price" >
              <el-input v-model="popupData.price" style="width:180px;padding-right:0;"/>
            </el-form-item>
          </el-col>
        </el-row>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="onSubmit">确定</el-button>
        <el-button @click="onClose">关闭</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
export default {
  name: 'Rent',
  data() {
    return {
      // 验证规则
      rules: {
        warehouseId: [
          { required: true, message: '请输入', trigger: 'change' }
        ],
        supplierId: [
          { required: true, message: '请输入', trigger: 'change' }
        ],
        price: [
          { required: true, message: '请输入', trigger: 'change' }
        ]
      },
      rulesB: {},
      formSearch: {
        supplierId: '',
        warehouseId: '',
        price: ''
      },
      table: {
        body: [],
        page: 1,
        pageSize: 15,
        paginationTotal: 0,
        autoHeight: 749,
        loading: false,
        currentPage: 1
      },
      popupData: {
        rentId: '',
        warehouseId: '',
        supplierId: '',
        price: ''
      },
      popupState: false,
      popupTitle: '',

      Export: {}
    }
  },
  mounted() {
    this.onSearch()
  },
  methods: {
    // 勾选
    selec() {},
    // 查询
    onSearch() {
      this.getTableData(1, 'page')
      this.table.currentPage = 1
    },
    // 查询请求
    onRequest() {
      const url = 'kaili-basic/v1/rents/queryRent'
      const params = {
        supplierId: this.formSearch.supplierId,
        warehouseId: this.formSearch.warehouseId
      }
      this.Export = Object.assign({}, params)
      params.page = this.table.page
      params.pageSize = this.table.pageSize
      this.table.loading = true
      this.getRequestParams(url, params).then((res) => {
        this.table.loading = false
        this.table.body = res.data.records
        this.table.paginationTotal = res.data.total
        this.table.pageSize = res.data.size
      })
    },

    // 新增按钮
    onOpen() {
      this.popupWay = true
      this.popupTitle = '新增客户'
      this.popupState = true
    },
    // 编辑
    onCompile(row) {
      this.popupWay = false
      this.popupState = true
      this.popupTitle = '编辑'
      for (const key in row) {
        this.popupData[key] = row[key]
      }
      this.$nextTick(() => {
        this.$refs['ruleForm'].clearValidate()
      })
    },

    // 表单提交
    onSubmit() {
      this.$refs['ruleForm'].validate((valid) => {
        if (valid) {
          if (this.popupWay) {
            const url = 'kaili-basic/v1/rents/addRent'
            const params = {
              price: this.popupData.price,
              supplierId: this.popupData.supplierId,
              warehouseId: this.popupData.warehouseId
            }
            this.postRequest(url, params).then((res) => {
              this.onRequest()
              this.onClose()
            })
          } else {
            const url = 'kaili-basic/v1/rents/updateRent'
            const params = {
              rentId: this.popupData.rentId,
              price: this.popupData.price,
              supplierId: this.popupData.supplierId,
              warehouseId: this.popupData.warehouseId
            }
            this.putRequest(url, params).then((res) => {
              this.onRequest()
              this.onClose()
            })
          }
        }
      })
    },

    // 刪除
    onDelete(row) {
      const this_ = this
      this.$confirm('确认删除吗？', '提示', {
        type: 'warning'
      }).then(() => {
        this_.table.loading = true
        const url = `kaili-basic/v1/rents/${row.rentId}/deleteRents`
        this.deleteRequest(url, '').then((res) => {
          this_.onRequest()
          this_.table.loading = false
          this_.$message({
            message: '删除成功',
            type: 'success'
          })
        })
      })
    },

    // 关闭弹窗
    onClose() {
      this.popupState = false
      for (const i in this.popupData) {
        this.popupData[i] = ''
      }
      this.$nextTick(() => {
        this.$refs['ruleForm'].clearValidate()
      })
    },

    // ========================
    // 导出
    onExport() {
      const arr = []
      for (const key in this.Export) {
        arr.push(key + '=' + this.Export[key])
      }
      const params = arr.join('&')
      window.location.href = `/api/kaili-basic/v1/rents/export?` + params
    },
    sizeChange(val) {
      this.getTableData(val, 'pageSize')
    },
    pageChange(val) {
      this.getTableData(val, 'page')
    },
    getTableData(val, flag) {
      if (flag === 'pageSize') {
        this.table.pageSize = val
      } else if (flag === 'page') {
        this.table.page = val
      }
      this.onRequest()
    }
  }
}
</script>
