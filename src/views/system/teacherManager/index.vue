<template>
  <div class="app-container calendar-list-container">
    <div class="filter-container">
      <el-input @keyup.enter.native="handleFilter" style="width: 100px;" class="filter-item" :placeholder="tableCol.tname" v-model="listQuery.tname">
      </el-input>
      <el-select clearable style="width: 90px" class="filter-item" v-model="listQuery.tsex" :placeholder="tableCol.tsex">
        <el-option v-for="item in sexOptions" :key="item" :label="item" :value="item">
        </el-option>
      </el-select>
      <el-select @change='handleFilter' style="width: 140px" class="filter-item" v-model="listQuery.sort">
        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key">
        </el-option>
      </el-select>
      <el-button class="filter-item" type="primary" v-waves icon="el-icon-search" @click="handleFilter">{{$t('table.search')}}</el-button>
      <el-button class="filter-item" style="margin-left: 10px;" @click="handleCreate" type="primary" icon="el-icon-edit">{{$t('table.add')}}</el-button>
      <el-button class="filter-item" type="primary" :loading="downloadLoading" v-waves icon="el-icon-upload2" @click="handleDownload">{{$t('table.export')}}</el-button>
      <upload-excel-component class="filter-item" v-waves @on-selected-file='selected'></upload-excel-component>
    </div>

    <el-table  :key='tableKey' :data="list" v-loading="listLoading" element-loading-text="给我一点时间" border fit highlight-current-row
      style="width: 100%">
      <el-table-column align="center" :label="tableCol.tno" width="65">
        <template slot-scope="scope">
          <span>{{scope.row.tno}}</span>
        </template>
      </el-table-column>
      <el-table-column width="100px" align="center" :label="tableCol.tname">
        <template slot-scope="scope">
          <span>{{scope.row.tname }}</span>
        </template>
      </el-table-column>
      <el-table-column width="50px" align="center" :label="tableCol.tsex">
        <template slot-scope="scope">
         <span>{{scope.row.tsex}}</span>
        </template>
      </el-table-column>
      <el-table-column width="200px" align="center" :label="tableCol.ttime">
        <template slot-scope="scope">
          <span>{{scope.row.ttime}}</span>
        </template>
      </el-table-column>
      <el-table-column min-width="110px"  align="center" :label="tableCol.tintroduce">
        <template slot-scope="scope">
          <el-alert title="" type="info" :closable="false">{{scope.row.tintroduce}}</el-alert>
        </template>
      </el-table-column>
      <el-table-column width="150px" align="center" :label="tableCol.twage">
        <template slot-scope="scope">
          <el-alert title="" type="warning" :closable="false">{{scope.row.twage}}</el-alert>
        </template>
      </el-table-column>
      <el-table-column align="center" label="操作" width="200" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="handleUpdate(scope.row)">{{$t('table.edit')}}</el-button>
          <el-button v-if="scope.row.status!='deleted'" size="mini" type="danger" @click="handleDelete(scope.row.tno)">{{$t('table.delete')}}
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-container">
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="listQuery.page" :page-sizes="[10,20,30, 50]" :page-size="listQuery.limit" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form :rules="rules" ref="dataForm" :model="temp" label-position="left" label-width="70px" style='width: 400px; margin-left:50px;'>
        <!-- <el-form-item :label="tableCol[0]" prop="sno">
          <el-select class="filter-item" v-model="temp.type" placeholder="Please select">
            <el-option v-for="item in  calendarTypeOptions" :key="item.key" :label="item.display_name" :value="item.key">
            </el-option>
          </el-select>
        </el-form-item> -->
        <el-form-item :label="tableCol.tname" prop="tname">
          <el-input v-model="temp.tname"></el-input>
        </el-form-item>
       
        <el-form-item :label="tableCol.tsex" prop="tsex">
           <el-select class="filter-item" v-model="temp.tsex" placeholder="请选择">
            <el-option v-for="item in  sexOptions" :key="item" :label="item" :value="item">
            </el-option>
          </el-select>
        </el-form-item>
       
        <el-form-item :label="tableCol.ttime" prop="ttime">
          <el-date-picker v-model="temp.ttime"  format="yyyy 年 MM 月 dd 日"  value-format="yyyy-MM-dd" placeholder="参与工作">
          </el-date-picker>
        </el-form-item>
        <el-form-item :label="tableCol.twage" prop="twage">
          <el-input v-model="temp.twage"></el-input>
        </el-form-item>

        <el-form-item :label="tableCol.ttel" prop="ttel">
          <el-input v-model="temp.ttel"></el-input>
        </el-form-item>
     
         <el-form-item :label="tableCol.tintroduce" prop="tintroduce">
          <el-input v-model="temp.tintroduce"
            type="textarea"
            :autosize="{ minRows: 5, maxRows: 7}"
            placeholder="请输入内容">
            </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">{{$t('table.cancel')}}</el-button>
        <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">{{$t('table.confirm')}}</el-button>
        <el-button v-else type="primary" @click="updateData">{{$t('table.confirm')}}</el-button>
      </div>
      
    </el-dialog>


  </div>
</template>

<script>
import { getTeacherData, createTeacher, updateTeacher, deleteTeacher } from '@/api/teacher'
import waves from '@/directive/waves' // 水波纹指令
import { parseTime } from '@/utils'
import UploadExcelComponent from '@/components/UploadExcel/index.vue'
import compare from '@/utils/compare'

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
      tableCol: {
        tno: '编号',
        tname: '名字',
        ttime: '参与工作',
        tintroduce: '简介',
        tsex: '性别',
        twage: '工资',
        ttel: '电话'
      },
      tableKey: 0,
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        tname: undefined,
        tsex: undefined,
        order: '+id'
      },
      classOptions: null,
      sexOptions: ['男', '女'],
      deptOptions: ['前端', '后端'],

      sortOptions: [{ label: '升序排序', key: '+id' }, { label: '降序排序', key: '-id' }],

      showReviewer: false,
      temp: {
        tno: undefined,
        tname: undefined,
        ttime: undefined,
        tintroduce: undefined,
        tsex: undefined,
        twage: undefined,
        ttel: undefined
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑',
        create: '新建'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [{ required: true, message: 'type is required', trigger: 'change' }],
        title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      },
      downloadLoading: false,
      tableData: null,
      tableHeader: null,
      //  tno: undefined,
      //   tname: undefined,
      //   ttime: undefined,
      //   tintroduce: undefined,
      //   tsex: undefined,
      //   twage: undefined
      tHeader: ['tno', 'tname', 'ttime', 'tintroduce', 'tsex', 'twage', 'ttel']
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
    selected(data) {
      this.tableData = data.results
      this.tableHeader = data.header
      console.log(this.tableData)
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
          delete data.results[j].tno
          createTeacher(data.results[j]).then(res => {
          })
        }
        this.getList()
        this.$message({
          message: '导入成功',
          type: 'success'
        })

        // console.log(this.list)
        // console.log(this.tableData)
        // console.log(this.list.length)
        // let j, len
        // for (j = 0, len = this.tableData.length; j < len; j++) {
        //   this.list.push(this.tableData[j])
        // }
        // this.list.concat(this.tableData)
        // console.log(this.list.length)
      }
    },
    getList() {
      this.listLoading = true
      getTeacherData(this.listQuery).then(response => {
        this.list = response.data.items
        this.total = response.data.total
        this.listLoading = false
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
        tno: undefined,
        tname: undefined,
        ttime: undefined,
        tintroduce: undefined,
        tsex: undefined,
        twage: undefined,
        ttel: undefined
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
          createTeacher(this.temp).then(res => {
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
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      console.log(this.temp)
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
          updateTeacher(tempData).then((res) => {
            if (res.data.data === 'success') {
              this.dialogFormVisible = false
              this.getList()
              this.$notify({
                title: '成功',
                message: '更新成功',
                type: 'success',
                duration: 2000
              })
            } else {
              this.$notify({
                title: '失败',
                message: '更新失败',
                type: 'error',
                duration: 2000
              })
            }
          })
        }
      })
    },
    handleDelete(tno) {
      this.$confirm('此操作将永久删除该职工, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteTeacher({ 'tno': tno }).then(res => {
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

    // sno: undefined,
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
        const filterVal = ['tno', 'tname', 'ttime', 'tintroduce', 'tsex', 'twage', 'ttel']
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
