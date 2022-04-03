<template>
  <div class="directorysContainer paddingContainer">
    <el-card>
      <!-- 面包导航 -->
      <el-breadcrumb separator="/" v-if="breadShow">
        <el-breadcrumb-item>学科</el-breadcrumb-item>
        <el-breadcrumb-item>{{subjectName}}</el-breadcrumb-item>
        <el-breadcrumb-item>
          <a href="/">目录管理</a>
        </el-breadcrumb-item>
      </el-breadcrumb>
      <!-- 面包导航结束  -->
      <el-row>
        <el-col :span="20">
          <el-form
            ref="form"
            :model="tableDatas"
            label-width="80px"
            :inline="true"
            class="directorysForm"
          >
            <el-form-item label="目录名称">
              <el-input v-model="tableDatas.directoryName"></el-input>
            </el-form-item>
            <el-form-item label="状态">
              <el-select v-model="tableDatas.state" placeholder="请选择">
                <el-option
                  v-for="(item, index) in statusOption"
                  :label="item.label"
                  :value="item.value"
                  :key="index"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button @click="clearDirectory">清除</el-button>
              <el-button type="primary" @click="searchDirectory">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="4" right>
          <!-- 返回按钮 -->
          <el-button type="text" v-if="breadShow" @click="$router.go(-1)">
            <i class="el-icon-back"></i>
            <span>返回学科</span>
          </el-button>
          <!-- 新增目录按钮 -->
          <el-button
            type="success"
            icon="el-icon-edit"
            @click="addDirectoryDialog"
            class="addDirectory"
          >新增目录</el-button>

          <!-- 新增对话框的事件 -->
          <el-dialog
            title="新增目录"
            :visible.sync="addDirectoryDialogVisible"
            width="30%"
            center
            @close="addDialogClosed"
          >
            <el-form
              :model="addtableData"
              :rules="addFormRules"
              ref="addFormRef"
              label-width="100px"
              class="demo-ruleForm"
            >
              <el-form-item label="所属学科">
                <el-select v-model="addtableData.subjectID" placeholder="请选择" class="select">
                  <el-option
                    v-for="(item, index) in allSubjectData"
                    :key="index"
                    :label="item.subjectName"
                    :value="item.id"
                  ></el-option>
                </el-select>
              </el-form-item>
              <el-form-item label="目录名称" prop="directoryName">
                <el-input v-model="addtableData.directoryName" placeholder="请输入目录名称"></el-input>
              </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
              <el-button @click="addDirectoryDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addDirectoryLists">确 定</el-button>
            </span>
          </el-dialog>
        </el-col>
      </el-row>

      <el-alert :title="alertInfo" type="info" show-icon :closable="false"></el-alert>

      <!-- 列表模块 -->
      <el-table
        border
        fit
        ref="singleTable"
        :data="tableData"
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column prop="id" label="序号" type="index" width="50"></el-table-column>
        <el-table-column prop="subjectName" label="所属学科" width="120"></el-table-column>
        <el-table-column prop="directoryName" label="目录名称" width="120"></el-table-column>
        <el-table-column prop="username" label="创建者"></el-table-column>

        <el-table-column prop="addDate" label="创建日期">
          <template
            slot-scope="scope"
          >{{ scope.row.addDate | parseTimeByString('{y}-{m}-{d} {h}:{i}:{s}') }}</template>
        </el-table-column>
        <el-table-column prop="totals" label="面试题数量"></el-table-column>
        <el-table-column prop="state" label="状态">
          <template slot-scope="scope">
            <span v-if="scope.row.state === 1">已启用</span>
            <span v-else>已禁用</span>
          </template>
        </el-table-column>
        <el-table-column prop label="操作">
          <template slot-scope="scope">
            <el-button
              type="text"
              class="showbtn"
              :row-key="scope.row.id"
              @click="clickDisabled(scope.row)"
            >
              {{
              scope.row.state ? '禁用' : '启用'
              }}
            </el-button>
            <el-button
              type="text"
              @click="editDirectorys(scope.row.id)"
              :disabled="scope.row.state === 1 ? true : false"
            >修改</el-button>
            <el-button
              type="text"
              :disabled="scope.row.state === 1 ? true : false"
              @click="deleteDirectorys(scope.row)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 修改对话框的事件 -->
      <el-dialog
        title="修改目录"
        :visible.sync="editDirectoryDialogVisible"
        width="30%"
        center
        @close="editDirectorysClosed"
      >
        <el-form
          :model="edittabledata"
          :rules="editFormrules"
          ref="editFormRef"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="所属学科" prop="subjectID">
            <el-select class="select" v-model="edittabledata.subjectID" placeholder="请选择">
              <el-option
                v-for="(item, index) in allSubjectData"
                :label="item.subjectName"
                :value="item.id"
                :key="index"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="目录名称" prop="directoryName">
            <el-input v-model="edittabledata.directoryName"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDirectoryDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editDirectorysInfo">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 分页模块 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="tableDatas.page"
        :page-sizes="[5, 10, 20, 50]"
        :page-size="tableDatas.pagesize"
        background
        :total="total"
        layout="prev, pager, next, total, sizes, jumper"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
import { list as comList } from '@/api/hmmm/subjects.js'
import { list, add, update, remove, changeState, detail } from '@/api/hmmm/directorys.js'
export default {
  name: 'directorys',
  data () {
    return {
      statusValue: [],
      statusOption: [
        {
          value: '1',
          label: '启用'
        },
        {
          value: '0',
          label: '禁用'
        }
      ],

      form: {
        name: '',
        region: '',
        directoryName: ''
      },
      // 目录列表数据
      tableData: [],
      tableDatas: {
        // 当前页码
        page: 1,
        // 每页显示多少条
        pagesize: 10,
        directoryName: null,
        state: null
      },

      // 所有学科数据对象
      allSubjectData: [],
      // 新增目录数据
      addtableData: {
        subjectID: '',
        directoryName: ''
      },
      // 修改目录数据
      edittabledata: {
        subjectID: '',
        directoryName: ''
      },
      editFormrules: {},
      editFormRef: {},
      addFormRules: {
        directoryName: [{ required: true, message: '请输入目录名称', trigger: 'blur' }]
      },

      // 总的数据条数
      total: 0,
      // 控制新增目录框的显示与隐藏
      addDirectoryDialogVisible: false,
      // 控制修改目录框的显示与隐藏
      editDirectoryDialogVisible: false,

      // 提示信息
      alertInfo: '',
      // 目录状态
      showStatus: false,
      state: '',
      breadShow: false,
      subjectName: ''
    }
  },

  created () {
    // 获取学科数据
    this.getSubjectList()
    // 目录列表
    this.directorysLists()
    // 新增目录
    // this.addDirectoryLists()
    // 修改目录
    // this.edittabledataLists()
    // 删除目录
  },

  watch: {
    $route: function () {
      console.log(this.$route.query)
    }
  },

  mounted () {
    this.tableDatas.subjectID = this.$route.query.id
    this.subjectName = this.$route.query.subjectName

    if (this.$route.query.id) {
      this.breadShow = true
    }
  },

  methods: {
    // 获取学科列表
    async getSubjectList () {
      const params = {
        pagesize: 50
      }
      const { data } = await comList(params)
      // console.log(data);
      this.allSubjectData = data.items
    },
    // 监听搜索事件
    searchDirectory () {
      // directorysLists(this.queryInfo)
      this.directorysLists()
    },
    // 清除事件
    clearDirectory () {
      this.tableDatas.state = null
      this.tableDatas.directoryName = ''
      this.directorysLists()
    },
    // 获取目录列表
    async directorysLists () {
      const { data } = await list(this.tableDatas)
      this.tableData = data.items
      this.total = data.counts
      this.alertInfo = `数据一共 ${data.counts} 条`
    },
    // 新增目录
    addDirectoryLists () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        await add(this.addtableData)
        this.addDirectoryDialogVisible = false
        this.directorysLists()
      })
    },
    // 监听新增目录对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
      this.addtableData.subjectID = null
      this.addtableData.directoryName = null
    },

    addDirectoryDialog () {
      this.addDirectoryDialogVisible = true
      this.getSubjectList()
    },
    // 每页显示多少条
    handleSizeChange (newSize) {
      this.tableDatas.pagesize = newSize
      this.directorysLists()
    },
    // 当前页码
    handleCurrentChange (newPage) {
      this.tableDatas.page = newPage
      this.directorysLists()
    },

    // 修改目录 根据id修改当前行的数据
    async editDirectorys (id) {
      const { data: res } = await detail({ id: id })
      //  console.log(res);
      this.edittabledata = res
      //  console.log(this.edittabledata);
      this.editDirectoryDialogVisible = true
    },
    // 点击确定按钮修改信息
    editDirectorysInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        await update(this.edittabledata)
        this.editDirectoryDialogVisible = false
        this.directorysLists()
        this.$message.success('修改信息成功！')
      })
    },
    // 监听对话框的关闭事件
    editDirectorysClosed () {
      // 重置表单
      this.$refs.editFormRef.resetFields()
    },

    // 目录状态
    async clickDisabled (directory) {
      await changeState({
        id: directory.id,
        state: directory.state === 1 ? 0 : 1
      })
      this.$message.success('操作成功！')
      directory.state = directory.state === 1 ? 0 : 1
    },
    // 删除功能
    async deleteDirectorys (obj) {
      const confirmRes = await this.$confirm('此操作将永久删除该目录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 删除成功 掉接口
      if (confirmRes === 'confirm') {
        await remove(obj)
        this.$message.success('删除成功')
        this.directorysLists()
      }
    }
  }
}
</script>
<style lang="scss" scoped>
.el-alert {
  width: 100%;
  padding: 8px 16px;
  margin-bottom: 15px;
  border-radius: 4px;
  background-color: #f4f4f5;
}
.el-pagination {
  margin-top: 20px;
  padding: 2px 5px;
  text-align: right;
}
.select {
  width: 100%;
}
.el-breadcrumb {
  margin-bottom: 10px;
}
</style>
