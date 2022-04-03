<template>
  <div class="subject-container paddingContainer">
    <el-card shadow="never">
      <!-- 学科名称搜索 -->
      <el-form :model="subjectQueryInfo" ref="subjectQueryInfoRef">
        <el-form-item label="学科名称" prop="kew">
          <el-input
            style="width: 200px;"
            :placeholder="$t('table.search')"
            class="filter-item"
            v-model="subjectQueryInfo.subjectName"
            clearable
            @clear="getSubjectList"
          ></el-input>
          <el-button class="filter-item" size="small" @click="clearSearchValue">清除</el-button>
          <el-button class="filter-item" size="small" type="primary" @click="getSubjectOnClick">搜索</el-button>
          <el-button
            class="filter-item fr"
            size="small"
            style="margin-left: 10px;"
            type="success"
            icon="el-icon-edit"
            @click="showAddSubjectForm"
          >新增学科</el-button>
        </el-form-item>
      </el-form>
      <el-alert :title="alertText" type="info" class="alert" :closable="false" show-icon></el-alert>
      <!-- end -->
      <!--学科数据 -->
      <el-table
        :data="subjectListData"
        row-key="false"
        fit
        highlight-current-row
        style="width: 100%"
        border
      >
        <el-table-column label="#" type="index"></el-table-column>
        <!-- <el-table-column align="center" label="序号" prop="id"></el-table-column> -->
        <el-table-column :label="$t('table.subjectName')">
          <template slot-scope="scope">
            <span>{{scope.row.subjectName}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.creator')">
          <template slot-scope="scope">
            <span>{{scope.row.username}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.creatdate')">
          <template slot-scope="scope">
            <span>{{scope.row.addDate|parseTimeByString("{y}-{m}-{d} {h}:{i}:{s}")}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.showinfront')">
          <template slot-scope="scope">
            <span>{{scope.row.isFrontDisplay?'是':'否'}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.subdirectory')">
          <template slot-scope="scope">
            <span>{{scope.row.twoLevelDirectory}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.tag')">
          <template slot-scope="scope">
            <span>{{scope.row.tags}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.titleNumber')">
          <template slot-scope="scope">
            <span>{{scope.row.totals}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.actions')" min-width="150" class="action-btn">
          <template slot-scope="scope">
            <el-button
              type="text"
              size="medium"
              @click="subjectClassficate(scope.row)"
            >{{$t('table.subjectClassification')}}</el-button>
            <el-button
              type="text"
              size="medium"
              @click="subjectTags(scope.row)"
            >{{$t('table.subjectTag')}}</el-button>
            <el-button
              type="text"
              size="medium"
              @click="showEditSubject(scope.row)"
            >{{$t('table.modify')}}</el-button>
            <el-button
              type="text"
              size="medium"
              @click="removeSubject(scope.row)"
            >{{$t('table.delete')}}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="subjectQueryInfo.page"
          :page-sizes="[3,5, 8, 10]"
          :page-size="subjectQueryInfo.pagesize"
          layout="prev, pager, next,sizes, jumper"
          :total="total"
        ></el-pagination>
      </div>
    </el-card>
    <!-- 新增用户对话框 -->
    <el-dialog
      title="新增学科"
      :visible.sync="addSubjectFormVisible"
      width="400px"
      @close="clearAddSubjectInput"
    >
      <el-form
        :model="addSubjectForm"
        :rules="addSubjectFormRule"
        ref="addSubjectFormRef"
        label-width="100px"
      >
        <el-form-item label="学科名称" required prop="subjectName">
          <el-input v-model="addSubjectForm.subjectName"></el-input>
        </el-form-item>
        <el-form-item label="是否显示" required prop="isFrontDisplay">
          <el-switch
            v-model="addSubjectForm.isFrontDisplay"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addSubjectFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addSubject">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 修改学科对话框 -->
    <el-dialog
      title="新增学科"
      :visible.sync="editSubjectFormVisible"
      width="400px"
      @close="cleareditSubjectInput"
    >
      <el-form
        :model="editSubjectForm"
        :rules="editSubjectFormRule"
        ref="editSubjectFormRef"
        label-width="100px"
      >
        <el-form-item label="学科名称" required prop="subjectName">
          <el-input v-model="editSubjectForm.subjectName"></el-input>
        </el-form-item>
        <el-form-item label="是否显示" required prop="isFrontDisplay">
          <el-switch
            v-model="editSubjectForm.isFrontDisplay"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editSubjectFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="editSubjectById">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { list, add, update, remove } from '@/api/hmmm/subjects.js'
// import { detail } from '@/api/base/users.js'
export default {
  name: 'subjects',
  data () {
    var checkText = (rule, value, callback) => {
      if (!value.trim()) {
        return callback(new Error('输入的是无效的空字符串'))
      } else {
        callback()
      }
    }
    return {
      subjectListData: [],
      alertText: '数据一共16条',
      subjectQueryInfo: {
        page: 1,
        pagesize: 10,
        subjectName: ''
      },
      total: 0,
      addSubjectFormVisible: false,
      addSubjectForm: {
        subjectName: '',
        isFrontDisplay: true
      },
      addSubjectFormRule: {
        subjectName: [{ min: 1, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' }, { validator: checkText, trigger: 'blur' }]
      },
      editSubjectForm: {
        subjectName: '',
        isFrontDisplay: true
      },
      editSubjectFormRule: {
        subjectName: [{ min: 1, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' }, { validator: checkText, trigger: 'blur' }]
      },
      editSubjectFormVisible: false
    }
  },
  created () {
    this.getSubjectList()
  },
  mounted () {

  },
  methods: {
    //  获取文章表格数据
    async getSubjectList () {
      try {
        const { data: res } = await list(this.subjectQueryInfo)
        this.subjectListData = res.items
        // console.log(this.subjectListData)
        console.log(res)
        this.alertText = `数据一共${res.counts}条`
        this.total = res.counts
      } catch (err) {
        this.$message.error('获取学科数据失败！')
      }
    },
    handleCurrentChange (val) {
      this.subjectQueryInfo.page = val
      this.getSubjectList()
    },
    handleSizeChange (val) {
      // console.log(val, '-----')
      this.subjectQueryInfo.pagesize = val
      this.getSubjectList()
    },
    // 清空搜索框
    clearSearchValue () {
      this.subjectQueryInfo.subjectName = ''
      this.getSubjectList()
    },
    // 点击搜索获取学科列表
    getSubjectOnClick () {
      this.subjectQueryInfo.subjectName = this.subjectQueryInfo.subjectName.trim()
      this.getSubjectList()
    },
    // 弹出添加学科对话框
    showAddSubjectForm () {
      this.addSubjectFormVisible = true
    },
    addSubject () {
      this.$refs.addSubjectFormRef.validate(async valid => {
        if (valid) {
          try {
            await add(this.addSubjectForm)
            this.$message.success('添加学科成功！')
            this.addSubjectFormVisible = false
            this.getSubjectList()
          } catch (err) {
            this.$message.error('添加学科失败！')
            this.addSubjectFormVisible = false
          }
        } else {
          this.$message({
            message: '学科名不合法,请输入！',
            type: 'warning'
          })
          return false
        }
      })
    },
    clearAddSubjectInput () {
      this.$refs.addSubjectFormRef.resetFields()
    },
    cleareditSubjectInput () {
      this.$refs.editSubjectFormRef.resetFields()
    },
    showEditSubject (row) {
      this.editSubjectFormVisible = true
      this.editSubjectForm.subjectName = row.subjectName
      this.editSubjectForm.isFrontDisplay = row.isFrontDisplay
      this.editSubjectForm.id = row.id
    },
    // 修改学科信息
    editSubjectById () {
      this.$refs.editSubjectFormRef.validate(async valid => {
        if (valid) {
          try {
            await update(this.editSubjectForm)
            this.$message.success('更新学科信息成功！')
            this.editSubjectFormVisible = false
            this.getSubjectList()
          } catch (err) {
            this.$message.error('更新学科信息失败！')
            this.editSubjectFormVisible = false
          }
        } else {
          this.$message({
            message: '学科名不合法,请重新输入！',
            type: 'warning'
          })
          return false
        }
      })
    },
    // 删除学科
    removeSubject (row) {
      this.$confirm('此操作将永久删除学科, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        remove(row).then(res => {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          this.getSubjectList()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    // 学科分类按钮功能
    subjectClassficate (row) {
      console.log(row)
      this.$router.push(`/subjects/directorys?id=${row.id}&subjectName=${row.subjectName}`)
    },
    // 学科目录按钮功能
    subjectTags (row) {
      this.$router.push(`/subjects/tags?id=${row.id}&subjectName=${row.subjectName}`)
    }
  }
}
</script>

<style scoped lang='scss'>
.subject-name {
  font-weight: 700;
  color: #646466;
  margin-right: 10px;
}
.filter-item {
  margin-left: 10px;
}
.pagination {
  margin-top: 18px;
  margin-bottom: 18px;
  float: right;
}
::v-deep .el-dialog__body {
  padding-left: 0;
}
::v-deep .el-form-item__label {
  text-align: unset;
  padding: 0 0 0 20px;
}
</style>
