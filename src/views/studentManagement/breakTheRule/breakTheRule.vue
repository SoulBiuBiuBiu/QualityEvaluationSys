<template>
  <div class="app-container calendar-list-container">
    <el-card>
    <div slot="header" class="clearfix">
      <span style="font-size:25px">违纪管理</span>
    </div>
    <div class="filter-container">
      <el-date-picker class="filter-item"
        v-model="listQuery.time"
        type="date"
        format="yyyy-MM-dd"
        placeholder="选择日期时间"
        value-format="yyyy-MM-dd">
      </el-date-picker>
      <el-input @keyup.enter.native="handleFilter" style="width: 100px;" class="filter-item" placeholder="请输入学号" v-model="listQuery.sid">
      </el-input>
      <el-select clearable style="width: 120px" class="filter-item" v-model="listQuery.status" :placeholder="tableCol.status">
        <el-option v-for="item in statusOptions" :key="item" :label="item" :value="item">
        </el-option>
      </el-select>
      <el-select @change='handleFilter' style="width: 140px" class="filter-item" v-model="listQuery.sort" :placeholder="tableCol.sort">
        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key">
        </el-option>
      </el-select>
      <el-button class="filter-item" type="primary" v-waves icon="el-icon-search" @click="handleFilter">{{$t('table.search')}}</el-button>
      <el-button class="filter-item" style="margin-left: 10px;" @click="handleCreate" type="primary" icon="el-icon-edit">{{$t('table.add')}}</el-button>
      <el-button class="filter-item" type="primary" :loading="downloadLoading" v-waves icon="el-icon-upload2" @click="handleDownload">{{$t('table.export')}}</el-button>
      <upload-excel-component class="filter-item" v-waves @on-selected-file='selected'></upload-excel-component>
    </div>

    <el-table  :key='tableKey' :data="list" border fit highlight-current-row
      style="width: 100%">
      <el-table-column width="100px"  align="center" :label="tableCol.time">
        <template slot-scope="scope">
          <span >{{scope.row.time}}</span>
        </template>
      </el-table-column>
      <el-table-column width="80px" align="center" :label="tableCol.sname">
        <template slot-scope="scope">
          <span>{{scope.row.sname }}</span>
        </template>
      </el-table-column>
      <el-table-column width="80px" align="center" :label="tableCol.sclass">
        <template slot-scope="scope">
          <span>{{scope.row.sclass}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" :label="tableCol.sid" width="80">
        <template slot-scope="scope">
          <span>{{scope.row.sid}}</span>
        </template>
      </el-table-column>
      <el-table-column min-width="150px" label="违纪内容">
        <template slot-scope="scope">
          <el-alert title="" type="error" :closable="false">{{scope.row.content}}</el-alert>
        </template>
      </el-table-column>
      <el-table-column align="center" :label="tableCol.status" width="95">
        <template slot-scope="scope">
          <el-tag type="danger">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
     
      <el-table-column align="center" label="操作" width="200" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="handleUptime(scope.row)">{{$t('table.edit')}}</el-button>
          <el-button v-if="scope.row.status!='deleted'" size="mini" type="danger" @click="handleDelete(scope.row.bid)">{{$t('table.delete')}}
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-container">
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="listQuery.page" :page-sizes="[10,20,30, 50]" :page-size="listQuery.limit" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

    <el-dialog :content="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form :rules="rules" ref="dataForm" :model="temp" label-position="left" label-width="70px" style='width: 400px; margin-left:50px;'>
       
        

        <el-form-item :label="tableCol.sname" prop="sname">
          <el-autocomplete
          class="inline-input"
          v-model="temp.sname"
          :fetch-suggestions="querySearch"
          placeholder="请输入名字"
          :trigger-on-focus="false"
          @select="handleSelect">
          </el-autocomplete>
        </el-form-item>
       <el-form-item :label="tableCol.sid" prop="sid">
          <el-input v-model="temp.sid"></el-input>
        </el-form-item>
        
        <el-form-item :label="tableCol.sclass" prop="sclass">
          <el-input v-model="temp.sclass"></el-input>
        </el-form-item>
         <el-form-item :label="tableCol.status" prop="status">
           <el-select class="filter-item" v-model="temp.status" placeholder="请选择">
            <el-option v-for="item in  statusOptions" :key="item" :label="item" :value="item">
            </el-option>
          </el-select>
        </el-form-item>
         <el-form-item :label="tableCol.time" prop="time">
          <el-date-picker class="filter-item"
            v-model="temp.time"
            type="date"
            format="yyyy-MM-dd"
            placeholder="选择日期时间"
            value-format="yyyy-MM-dd">
          </el-date-picker>
        </el-form-item>
        <el-form-item :label="tableCol.content" prop="content">
          <el-input v-model="temp.content"
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 4}">
          </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">{{$t('table.cancel')}}</el-button>
        <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">{{$t('table.confirm')}}</el-button>
        <el-button v-else type="primary" @click="uptimeData">{{$t('table.confirm')}}</el-button>
      </div>
      
    </el-dialog>

    </el-card>
  </div>
</template>

<script>
import { fetchListBreakRule, createBreakRole, deleteBreakRole, updateBreakRole } from '@/api/breakRole'
import waves from '@/directive/waves' // 水波纹指令
import { parseTime } from '@/utils'
import UploadExcelComponent from '@/components/UploadExcel/index.vue'
import compare from '@/utils/compare'
import { getStudentAll } from '@/api/student'

export default {
  name: 'complexTable',
  directives: {
    waves
  },
  components: {
    UploadExcelComponent
  },
  data() {
    return {
      // '学号', '姓名', '性别', '班级', '生日', '地址', '系别', '入学时间', '操作', '排序规则'
      restaurants: [],
      tableCol: {
        sid: '学号',
        sname: '姓名',
        sclass: '班级',
        time: '时间',
        content: '内容',
        status: '处分程度',
        sort: '排序方式'
      },
      oldtemp: null,
      tableKey: 0,
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        sid: undefined,
        time: undefined,
        sort: '-id',
        sclass: undefined
      },
      classOptions: ['vue.js'],
      sexOptions: ['男', '女'],
      statusOptions: ['警告', '严重警告', '处分', '严重处分'],
      sortOptions: [{ label: '升序排序', key: '+id' }, { label: '降序排序', key: '-id' }],

      showReviewer: false,
      temp: {
        sid: undefined,
        sname: undefined,
        sclass: undefined,
        time: undefined,
        content: undefined,
        status: undefined
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        uptime: '编辑',
        create: '新建'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [{ required: true, message: 'type is required', trigger: 'change' }],
        timestamp: [{ type: 'time', required: true, message: 'timestamp is required', trigger: 'change' }],
        content: [{ required: true, message: 'content is required', trigger: 'blur' }]
      },
      downloadLoading: false,
      tableData: null,
      tableHeader: null,
      // sid: undefined,
      // sname: undefined,
      // ssex: undefined,
      // sclass: undefined,
      // time: undefined,
      // reason: undefined,
      // content: undefined,
      // status: undefined
      tHeader: ['sid', 'sname', 'sclass', 'time', 'content', 'status']
    }
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    }

  },
  created() {
    this.getList()
  },
  methods: {
    querySearch(queryString, cb) {
      var restaurants = this.restaurants
      var results = queryString ? restaurants.filter(this.createFilter(queryString)) : restaurants
      // 调用 callback 返回建议列表的数据
      cb(results)
    },
    handleSelect(item) {
      this.temp.sid = item.address
    },
    createFilter(queryString) {
      return (restaurant) => {
        return (restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0)
      }
    },
    selected(data) {
      this.tableData = data.results
      this.tableHeader = data.header
      if (!compare(this.tableHeader, this.tHeader)) {
        this.$notify({
          title: '失败',
          message: '导入excel格式不对',
          type: 'failed',
          duration: 2000
        })
      } else {
        for (let j = 0; data.results[j] != null; j++) {
          // 去掉导入内容的主键
          createBreakRole(data.results[j]).then(res => {
          })
        }
        this.getList()
        this.$message({
          message: '导入成功',
          type: 'success'
        })
      }
    },
    getList() {
      getStudentAll().then(response => {
        const items = response.data.items
        for (let i = 0; i < items.length; i++) {
          this.restaurants.push({
            'value': '' + items[i].sname,
            'address': items[i].sid
          })
        }
        this.listLoading = true
        console.log(1)
        fetchListBreakRule(this.listQuery).then(response => {
          this.list = response.data.items
          this.total = response.data.total
          this.listLoading = false
        })
      })
    },
    handleFilter() {
      this.listQuery.page = 1
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

    resetTemp() {
      this.temp = {
        sid: undefined,
        sname: undefined,
        sclass: undefined,
        time: undefined,
        content: undefined,
        status: undefined
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValitime()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          createBreakRole(this.temp).then(res => {
            if (res.data.data === 'success') {
              this.getList()
              this.dialogFormVisible = false
              this.$notify({
                title: '成功',
                message: '创建成功',
                type: 'success',
                duration: 2000
              })
            } else {
              this.$notify({
                title: '失败',
                message: '创建失败',
                type: 'error',
                duration: 2000
              })
            }
          })
        }
      })
    },
    handleUptime(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.oldtemp = row
      console.log(this.oldtemp)
      console.log(this.temp)
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogStatus = 'uptime'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValitime()
      })
    },
    uptimeData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateBreakRole(tempData).then(() => {
            for (const v of this.list) {
              if (v.pid === this.temp.pid) {
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
    handleDelete(bid) {
      this.$confirm('此操作将永久删除该违纪记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteBreakRole({ 'bid': bid }).then(res => {
          if (res.data.data === 'success') {
            this.getList()
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
          } else {
            this.$message({
              type: 'info',
              message: '删除失败'
            })
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },

    // sid: undefined,
    //     sname: undefined,
    //     ssex: undefined,
    //     sclass: undefined,
    //     birth: undefined,
    //     saddress: undefined,
    //     sprofession: undefined,
    //     stime: undefined
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const filterVal = ['sid', 'sname', 'sclass', 'time', 'content', 'status']
        const data = this.formatJson(filterVal, this.list)
        excel.export_json_to_excel(this.tHeader, data, 'table-list')
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
