<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-date-picker
        v-model="dataForm.years"
        type="year"
        value-format="yyyy"
        placeholder="请选择年份">
      </el-date-picker>
      <el-select v-if="isAuth('biz:pdagency:down:list')"  clearable  v-model="dataForm.agencyId" placeholder="请选择机构">
        <el-option
          v-for="item in options"
          :key="item.id"
          :label="item.agencyName"
          :value="item.id">
        </el-option>
      </el-select>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('biz:pdenrollmentgoal:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <!--<el-button v-if="isAuth('sys:user:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>-->
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
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
        width="80"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="agencyName"
        align="center"
        label="机构">
      </el-table-column>
      <el-table-column
        prop="studentNum"
        header-align="center"
        align="center"
        label="预计招生人数">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="各渠道预计招生人数">
        <el-table-column
          prop="onlineNum"
          header-align="center"
          align="center"
          label="线上">
        </el-table-column>
        <el-table-column
          prop="pusnNum"
          header-align="center"
          align="center"
          label="地推">
        </el-table-column>
        <el-table-column
          prop="edcactionNum"
          header-align="center"
          align="center"
          label="教学部">
        </el-table-column>
        <el-table-column
          prop="studioNum"
          header-align="center"
          align="center"
          label="画室">
        </el-table-column>
        <el-table-column
          prop="otherNum"
          header-align="center"
          align="center"
          label="其他">
        </el-table-column>
      </el-table-column>

      <el-table-column
        prop="realSum"
        header-align="center"
        align="center"
        label="实际招生人数">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="各渠道实际招生人数">
        <el-table-column
          prop="realOnlineNum"
          header-align="center"
          align="center"
          label="线上">
        </el-table-column>
        <el-table-column
          prop="realPusnNum"
          header-align="center"
          align="center"
          label="地推">
        </el-table-column>
        <el-table-column
          prop="realEduNum"
          header-align="center"
          align="center"
          label="教学部">
        </el-table-column>
        <el-table-column
          prop="realStudioNum"
          header-align="center"
          align="center"
          label="画室">
        </el-table-column>
        <el-table-column
          prop="realOtherNum"
          header-align="center"
          align="center"
          label="其他">
        </el-table-column>
      </el-table-column>
      <el-table-column
        prop="enrollmentYear"
        header-align="center"
        align="center"
        label="年份">
        <template slot-scope="scope">
        {{ (scope.row.enrollmentYear).split("-")[0]}}
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('biz:pdenrollmentgoal:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="isAuth('biz:pdenrollmentgoal:delete')" type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
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
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './studentPlan-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          years: '',
          agencyId:''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        options: [{
          value: '选项1',
          label: '黄金糕'
        }, {
          value: '选项2',
          label: '双皮奶'
        }, {
          value: '选项3',
          label: '蚵仔煎'
        }, {
          value: '选项4',
          label: '龙须面'
        }, {
          value: '选项5',
          label: '北京烤鸭'
        }],
        value: '',
        value1: '',
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList();
      this.$http({
        url: this.$http.adornUrl('/biz/pdagency/down/list'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        this.options = data && data.code === 200 ? data.data : []
      })
    },
    methods: {
      // 获取数据列表
      getDataList () {
        console.log(this.value1)
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/biz/pdenrollmentgoal/list'),
          method: 'get',
          params: this.$http.adornParams({
            'pageNum': this.pageIndex,
            'pageSize': this.pageSize,
            'years': this.dataForm.years,
            'agencyId':this.dataForm.agencyId
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.dataList = data.data.list
            this.totalPage = data.data.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
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
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var userIds = id ? [id] : this.dataListSelections.map(item => {
          return item.userId
        })
        this.$confirm(`确认删除该条数据吗?删除后数据不可恢复`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/biz/pdenrollmentgoal/delete'),
            method: 'post',
            data: this.$http.adornData(userIds, false)
          }).then(({data}) => {
            if (data && data.code === 200) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      }
    }
  }
</script>
