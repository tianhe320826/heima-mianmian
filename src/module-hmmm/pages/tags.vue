<template>
  <div class="tags-container paddingContainer">
    <el-card>
      <!-- 面包导航 -->
      <el-breadcrumb separator="/" v-if="breadShow">
        <el-breadcrumb-item>学科</el-breadcrumb-item>
        <el-breadcrumb-item>{{queryInfo.tagName}}</el-breadcrumb-item>
        <el-breadcrumb-item>
          <a href="/">目录管理</a>
        </el-breadcrumb-item>
      </el-breadcrumb>
      <!-- 面包导航结束  -->
      <el-row class="row">
        <el-col class="col1">
          <el-form :inline="true" class="demo-form-inline" ref="clearFormRef" :model="queryInfo">
            <el-form-item label="目录名称">
              <el-input v-model="queryInfo.tagName"></el-input>
            </el-form-item>
            <el-form-item label="状态" class="statearea">
              <el-select v-model="queryInfo.state" placeholder="请选择">
                <el-option
                  v-for="(item, index) in statusOptions"
                  :key="index"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button @click="clearTag">清除</el-button>
              <el-button type="primary" @click="searchTags">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <!-- 返回按钮 -->
        <el-button type="text" v-if="breadShow" class="back-btn" @click="$router.go(-1)">
          <i class="el-icon-back"></i>
          <span>返回学科</span>
        </el-button>
        <el-col class="col2">
          <!-- 新增目录按钮 -->
          <el-button type="success" icon="el-icon-edit" @click="addTagsDialog">新增目录</el-button>
          <!-- 新增目录框 -->
          <el-dialog
            title="新增目录"
            :visible.sync="addTagsDialogVisible"
            width="30%"
            center
            @close="addDialogClosed"
          >
            <el-form
              :model="addTags"
              :rules="addFormrules"
              ref="addFormRef"
              label-width="100px"
              class="demo-ruleForm"
            >
              <el-form-item label="所属学科">
                <el-select v-model="addTags.subjectID" placeholder="请选择" class="select">
                  <el-option
                    v-for="(item, index) in allSubjectData"
                    :key="index"
                    :label="item.subjectName"
                    :value="item.id"
                  ></el-option>
                </el-select>
              </el-form-item>
              <el-form-item label="目录名称" prop="tagName">
                <el-input v-model="addTags.tagName" placeholder="请输入目录名称"></el-input>
              </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
              <el-button @click="addTagsDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addTagsLists">确 定</el-button>
            </span>
          </el-dialog>
        </el-col>
      </el-row>
      <el-alert :title="alertInfo" type="info" show-icon :closable="false"></el-alert>
      <!-- 目录表格 -->
      <el-table :data="tagsData" highlight-current-row style="width: 100%" fit border>
        <el-table-column prop="id" label="序号" type="index" width="50"></el-table-column>
        <el-table-column prop="subjectName" label="所属学科" width="120"></el-table-column>
        <el-table-column prop="tagName" label="标签名称" width="120"></el-table-column>
        <el-table-column prop="username" label="创建者"></el-table-column>
        <el-table-column prop="addDate" label="创建日期">
          <template
            slot-scope="scope"
          >{{ scope.row.addDate | parseTimeByString('{y}-{m}-{d} {h}:{i}:{s}') }}</template>
        </el-table-column>
        <el-table-column property="state" label="状态">
          <template slot-scope="scope">
            <span v-if="scope.row.state === 1">已启用</span>
            <span v-else>已禁用</span>
          </template>
        </el-table-column>
        <el-table-column property label="操作">
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
              @click="editTags(scope.row.id)"
              :disabled="scope.row.state === 1 ? true : false"
            >修改</el-button>
            <el-button
              type="text"
              :disabled="scope.row.state === 1 ? true : false"
              @click="deleteTags(scope.row)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 修改框的显示与隐藏 -->
      <el-dialog
        title="修改目录"
        :visible.sync="editTagsDialogVisible"
        width="30%"
        center
        @close="editTagsClosed"
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
                :label="item.subjectName"
                :value="item.id"
                v-for="(item, index) in allSubjectData"
                :key="index"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="目录名称" prop="tagName">
            <el-input v-model="edittabledata.tagName"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editTagsDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editTagsInfo">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 分页模块 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.page"
        :page-sizes="[5, 10, 20, 50]"
        :page-size="queryInfo.pagesize"
        :total="total"
        background
        layout="prev, pager, next, total, sizes, jumper"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
import { list as comList } from '@/api/hmmm/subjects.js'
import { list, changeState, add, remove, detail, update } from '@/api/hmmm/tags.js'
export default {
  name: 'tags',
  data () {
    return {
      statusValue: [],
      statusOptions: [
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
        tagName: ''
      },
      value: '',
      // 标签列表数据
      queryInfo: { page: 1, pagesize: 10, subjectID: null, tagName: '', state: null },

      tagsData: [],
      total: 0,

      // 所有学科数据对象
      allSubjectData: [],

      // 新增标签数据
      addTags: {
        subjectID: null,
        tagName: null
      },
      // 修改数据
      edittabledata: {
        subjectID: '',
        tagName: ''
      },
      ruleForm: {},
      // 提示信息
      alertInfo: '',
      // 新增标签弹框的显示与隐藏
      addTagsDialogVisible: false,
      editFormrules: {
        tagName: [{ required: true, message: '请输入目录名称', trigger: 'blur' }]
      },
      addFormrules: {
        tagName: [{ required: true, message: '请输入目录名称', trigger: 'blur' }]
      },
      // 修改目录弹框的显示与隐藏
      editTagsDialogVisible: false,
      breadShow: false,
      subjectName: ''
    }
  },
  created () {
    // 标签列表数据
    this.tagsLists()
    // 获取所属学科列表数据
    this.getSubjectList()
  },
  watch: {
    $route: function () {
      console.log(this.$route.query)
    }
  },
  mounted () {
    this.queryInfo.subjectID = this.$route.query.id
    console.log(this.$route.query)
    this.queryInfo.tagName = this.$route.query.subjectName
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
      // console.log(data)
      this.allSubjectData = data.items
    },
    // 点击新增 重新获取数据
    addTagsLists () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        await add(this.addTags)
        this.addTagsDialogVisible = false
        this.tagsLists()
      })
    },
    // 点击新增 获取学科列表
    addTagsDialog () {
      this.addTagsDialogVisible = true
      this.getSubjectList()
    },
    // 监听新增目录对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
      this.addTags.subjectID = null
    },

    // 每页显示多少条
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.tagsLists()
    },
    // 当前页码
    handleCurrentChange (newPage) {
      this.queryInfo.page = newPage
      this.tagsLists()
    },
    // 获取标签列表
    async tagsLists () {
      const data = await list(this.queryInfo)
      // console.log(data)
      this.tagsData = data.data.items
      this.total = data.data.counts
      this.alertInfo = `数据一共 ${data.data.counts} 条`
    },
    // 标签状态
    async clickDisabled (tags) {
      await changeState({
        id: tags.id,
        state: tags.state === 1 ? 0 : 1
      })
      this.$message.success('操作成功！')
      // tags.state = tags.state === 1 ? 0 : 1
      this.tagsLists()
    },
    // 删除标签
    async deleteTags (obj) {
      // console.log(obj);
      const result = await this.$confirm('此操作将永久删除该标签, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result === 'confirm') {
        await remove({ id: obj.id })
        this.$message.success('删除成功')
        this.tagsLists()
      } else {
        this.$message.info('取消删除')
      }
    },
    // 修改目录 根据id修改当前行的数据
    async editTags (id) {
      const { data: res } = await detail({ id: id })
      //  console.log(res);
      this.edittabledata = res
      //  console.log( this.edittabledata);
      this.editTagsDialogVisible = true
    },
    // 点击确定按钮修改信息
    editTagsInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        await update(this.edittabledata)
        this.editTagsDialogVisible = false
        this.tagsLists()
        this.$message.success('修改信息成功！')
      })
    },
    // 监听对话框的关闭事件
    editTagsClosed () {
      // 重置表单
      this.$refs.editFormRef.resetFields()
    },
    // 监听搜索事件
    searchTags () {
      this.tagsLists()
    },
    //  清除事件
    clearTag () {
      // console.log(this.$refs.clearFormRef)
      // this.$refs.clearFormRef.resetFields()
      this.queryInfo.state = null
      this.queryInfo.tagName = ''
      this.tagsLists()
    }
  }
}
</script>

<style scoped lang="scss">
.el-card {
  .row {
    display: flex;

    .col2 {
      flex: 1;
      justify-content: flex-end;
    }
    .statearea {
      margin-left: 40px;
    }
  }
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
    margin-bottom: 20px;
  }
  .back-btn {
    margin-right: 10px;
    position: relative;
    top: -10px;
  }
}
</style>
