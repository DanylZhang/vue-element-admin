<template>
  <div class="app-container">
    <expand-block tip="SQL编辑器">
      <div class="filter-container">
        <!--搜索条件: target url-->
        <el-input @keyup.enter.native="handleFilter" style="width: 350px;" class="filter-item"
                  placeholder="http://目标网址" v-model="listQuery.url">
        </el-input>
        <!--搜索条件: target regex-->
        <el-input style="width: 140px;" class="filter-item"
                  placeholder="校验正则" v-model="listQuery.regex">
        </el-input>
        <!--搜索条件：timeout-->
        <el-tooltip class="item" effect="dark" content="校验代理超时时间" :open-delay=800 placement="top">
          <el-input-number style="width: 100px;" class="filter-item" controls-position="right"
                           :min="1" :max="60" v-model="listQuery.timeout">
          </el-input-number>
        </el-tooltip>

        <el-button class="filter-item" type="primary" :loading="searchLoading" v-waves icon="el-icon-search"
                   @click="handleFilter">
          {{$t('table.search')}}
        </el-button>
        <el-button class="filter-item" style="margin-left: 10px;" @click="handleCreate" type="primary"
                   icon="el-icon-edit">{{$t('table.add')}}
        </el-button>
        <el-button class="filter-item" type="primary" :loading="downloadLoading" v-waves icon="el-icon-download"
                   @click="handleDownload">{{$t('table.export')}}
        </el-button>
      </div>

      <div class="editor-container" slot="expand">
        <sql-editor v-model="listQuery.where" ref="editor"></sql-editor>
      </div>
    </expand-block>

    <!--分页-->
    <div class="top-pagination-container">
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange"
                     :current-page="listQuery.page" :page-sizes="[10,20,30,50]" :page-size="listQuery.limit"
                     layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

    <!--表格-->
    <el-table :key='tableKey' :data="records" border fit highlight-current-row
              :default-sort="{prop: 'SPEED', order: 'ascending'}"
              @header-click="handleHeaderClick"
              v-loading="listLoading"
              :element-loading-text="listLoadingText"
              style="width: 100%;min-height:1000px;">
      <!--多选-->
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>

      <el-table-column v-for="(item,index) in fields" :key="index"
                       :prop="item.name" sortable="custom" show-overflow-tooltip
                       min-width="120px"
                       align="center"
                       :label="addSortSuffix(item.name,item.name)">
        <template slot-scope="scope">
          <el-tag v-if="item.name === 'TYPE' || item.name ==='IS_VALID'" :type="scope.row[index] | typeFilter">
            {{scope.row[index]}}
          </el-tag>
          <span v-else>
            {{scope.row[index]}}
          </span>
        </template>
      </el-table-column>

      <el-table-column fixed="right" align="center" :label="$t('table.actions')" min-width="260px"
                       class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="handleUpdate(scope.row)">{{$t('table.edit')}}</el-button>
          <el-button v-if="scope.row.status!='published'" size="mini" type="success"
                     @click="handleModifyStatus(scope.row,'published')">{{$t('table.publish')}}
          </el-button>
          <el-button v-if="scope.row.status!='deleted'" size="mini" type="danger"
                     @click="handleModifyStatus(scope.row,'deleted')">{{$t('table.delete')}}
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form :rules="rules" ref="dataForm" :model="temp" label-position="left" label-width="70px"
               style='width: 400px; margin-left:50px;'>
        <el-form-item :label="$t('table.type')" prop="type">
        </el-form-item>
        <el-form-item :label="$t('table.date')" prop="timestamp">
          <el-date-picker v-model="temp.timestamp" type="datetime" placeholder="Please pick a date">
          </el-date-picker>
        </el-form-item>
        <el-form-item :label="$t('table.title')" prop="title">
          <el-input v-model="temp.title"></el-input>
        </el-form-item>
        <el-form-item :label="$t('table.status')">
          <el-select class="filter-item" v-model="temp.status" placeholder="Please select">
            <el-option v-for="item in  statusOptions" :key="item" :label="item" :value="item">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item :label="$t('table.importance')">
          <el-rate style="margin-top:8px;" v-model="temp.importance" :colors="['#99A9BF', '#F7BA2A', '#FF9900']"
                   :max='3'></el-rate>
        </el-form-item>
        <el-form-item :label="$t('table.remark')">
          <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="Please input"
                    v-model="temp.remark">
          </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">{{$t('table.cancel')}}</el-button>
        <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">{{$t('table.confirm')}}</el-button>
        <el-button v-else type="primary" @click="updateData">{{$t('table.confirm')}}</el-button>
      </div>
    </el-dialog>

    <el-dialog title="Reading statistics" :visible.sync="dialogPvVisible">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel"></el-table-column>
        <el-table-column prop="pv" label="Pv"></el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">{{$t('table.confirm')}}</el-button>
      </span>
    </el-dialog>

  </div>
</template>
<script>
  import { createArticle, fetchPv, updateArticle } from '@/api/article'
  import waves from '@/directive/waves' // 水波纹指令
  import { parseTime } from '@/utils'
  import axios from 'axios'
  import SqlEditor from '../../components/SqlEditor/index'
  import ExpandBlock from '../../components/ExpandBlock/index'

  export default {
    name: 'complexTable',
    components: { SqlEditor, ExpandBlock },
    directives: {
      waves
    },
    data() {
      return {
        tableKey: 0,
        fields: null,
        records: null,
        total: null,
        searchLoading: false,
        listLoading: true,
        listLoadingText: '拼命加载中',
        listQuery: {
          page: 1,
          limit: 20,
          url: undefined,
          regex: undefined,
          timeout: 30,
          where: undefined,
          orderBy: [
            { SPEED: 'ascending' }
          ]
        },
        statusOptions: ['published', 'draft', 'deleted'],
        showReviewer: false,
        temp: {
          id: undefined,
          importance: 1,
          remark: '',
          timestamp: new Date(),
          title: '',
          type: '',
          status: 'published'
        },
        dialogFormVisible: false,
        dialogStatus: '',
        textMap: {
          update: 'Edit',
          create: 'Create'
        },
        dialogPvVisible: false,
        pvData: [],
        rules: {
          type: [{ required: true, message: 'type is required', trigger: 'change' }],
          timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
          title: [{ required: true, message: 'title is required', trigger: 'blur' }]
        },
        downloadLoading: false
      }
    },
    filters: {
      typeFilter(type) {
        const typeMap = {
          http: 'success',
          https: 'primary',
          socks: 'danger'
        }
        return typeMap[type]
      }
    },
    created() {
      this.getSqlHint()
      this.getList()
    },
    methods: {
      getList() {
        // 更新等待时长
        let cost = 0
        this.listLoadingText = `拼命搜索中, ${cost}s...`
        const interval = setInterval(() => {
          cost++
          this.listLoadingText = `拼命搜索中, ${cost}s...`
        }, 1000)

        // 打开loading
        this.listLoading = true
        this.searchLoading = true

        console.log(JSON.stringify(this.listQuery))
        axios.post('/api/proxy/table', this.listQuery)
          .then(response => {
            response = response.data
            console.log(response)
            if (response.code === 0) {
              this.fields = response.data.fields
              this.records = response.data.records
              this.total = response.data.total
            } else {
              this.$message({
                showClose: true,
                message: response.msg,
                type: 'warning',
                center: true
              })
            }
          }).finally(() => {
            this.listLoading = false
            this.searchLoading = false
            clearInterval(interval)
          })
      },
      getSqlHint() {
        axios.get('/api/proxy/getSqlHint')
          .then(response => {
            response = response.data
            if (response.code === 0) {
              this.$refs.editor.setOption('hintOptions', response.data)
            }
          })
      },
      handleFilter() {
        this.listQuery.page = 1
        if (this.listQuery.url) {
          this.listQuery.orderBy.splice(0, this.listQuery.orderBy.length, { speed: 'ascending' })
        }
        this.getList()
      },
      handleSizeChange(val) {
        this.listQuery.limit = val
        this.getList()
      },
      handleCurrentChange(val) {
        this.listQuery.page = val
        this.getList()
      },
      handleHeaderClick(column, event) {
        if (column.type === 'selection') {
          return
        }
        const currentField = column.property
        const currentOrder = column.order
        const index = this._getOrderByIndex(this.listQuery.orderBy, currentField)
        // 有排序
        if (currentOrder) {
          // orderBy数组中已有则改序
          if (index > -1) {
            this.listQuery.orderBy[index][currentField] = currentOrder
          } else { // orderBy数组中没有则添加
            const order = {}
            order[currentField] = currentOrder
            this.listQuery.orderBy.push(order)
          }
        } else { // 无排序则从orderBy中删除此field
          if (index > -1) {
            this.listQuery.orderBy.splice(index, 1)
          }
        }
        this.listQuery.url = undefined
        this.getList()
      },
      _getOrderByIndex(orderByArr, field) {
        for (let i = 0; i < orderByArr.length; i++) {
          for (const key in orderByArr[i]) {
            if (key === field) {
              return i
            }
          }
        }
        return -1
      },
      addSortSuffix(label, prop) {
        let result = label
        const labelSuffix = this._getOrderByIndex(this.listQuery.orderBy, prop) + 1
        if (labelSuffix > 0) {
          result += ' ' + labelSuffix
        }
        return result
      },
      handleModifyStatus(row, status) {
        this.$message({
          message: '操作成功',
          type: 'success'
        })
        row.status = status
      },
      resetTemp() {
        this.temp = {
          id: undefined,
          importance: 1,
          remark: '',
          timestamp: new Date(),
          title: '',
          status: 'published',
          type: ''
        }
      },
      handleCreate() {
        this.resetTemp()
        this.dialogStatus = 'create'
        this.dialogFormVisible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].clearValidate()
        })
      },
      createData() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
            this.temp.author = 'vue-element-admin'
            createArticle(this.temp).then(() => {
              this.list.unshift(this.temp)
              this.dialogFormVisible = false
              this.$notify({
                title: '成功',
                message: '创建成功',
                type: 'success',
                duration: 2000
              })
            })
          }
        })
      },
      handleUpdate(row) {
        this.temp = Object.assign({}, row) // copy obj
        this.temp.timestamp = new Date(this.temp.timestamp)
        this.dialogStatus = 'update'
        this.dialogFormVisible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].clearValidate()
        })
      },
      updateData() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            const tempData = Object.assign({}, this.temp)
            tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
            updateArticle(tempData).then(() => {
              for (const v of this.list) {
                if (v.id === this.temp.id) {
                  const index = this.list.indexOf(v)
                  this.list.splice(index, 1, this.temp)
                  break
                }
              }
              this.dialogFormVisible = false
              this.$notify({
                title: '成功',
                message: '更新成功',
                type: 'success',
                duration: 2000
              })
            })
          }
        })
      },
      handleDelete(row) {
        this.$notify({
          title: '成功',
          message: '删除成功',
          type: 'success',
          duration: 2000
        })
        const index = this.list.indexOf(row)
        this.list.splice(index, 1)
      },
      handleFetchPv(pv) {
        fetchPv(pv).then(response => {
          this.pvData = response.data.pvData
          this.dialogPvVisible = true
        })
      },
      handleDownload() {
        this.downloadLoading = true
        import('@/vendor/Export2Excel').then(excel => {
          const tHeader = ['timestamp', 'title', 'type', 'importance', 'status']
          const filterVal = ['timestamp', 'title', 'type', 'importance', 'status']
          const data = this.formatJson(filterVal, this.list)
          excel.export_json_to_excel({
            header: tHeader,
            data,
            filename: 'table-list'
          })
          this.downloadLoading = false
        })
      },
      formatJson(filterVal, jsonData) {
        return jsonData.map(v => filterVal.map(j => {
          if (j === 'timestamp') {
            return parseTime(v[j])
          } else {
            return v[j]
          }
        }))
      }
    }
  }
</script>
