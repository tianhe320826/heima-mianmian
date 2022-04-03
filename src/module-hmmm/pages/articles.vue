<template>
  <div class="article-container">
    <el-card class="paddingContainer">
      <el-form :model="articleListQueryInfo" ref="subjectQueryInfoRef">
        <el-form-item label="关键字" prop="kew">
          <el-input
            style="width: 200px;"
            class="filter-item"
            v-model="articleListQueryInfo.keyword"
            clearable
          ></el-input>
          <span>
            <label for="statusLabel" style="color:#606266;padding-left:40px;padding-right:10px">状态</label>
            <el-select
              clearable
              id="statusLabel"
              placeholder="请选择"
              v-model="articleListQueryInfo.state"
            >
              <el-option label="启用" :value="1"></el-option>
              <el-option label="禁用" :value="0"></el-option>
            </el-select>
          </span>
          <el-button class="filter-item" size="small" @click="clearInput">清除</el-button>
          <el-button class="filter-item" size="small" type="primary" @click="searchArticle">搜索</el-button>
          <el-button
            class="filter-item fr"
            size="small"
            style="margin-left: 10px;"
            type="success"
            icon="el-icon-edit"
            @click="addArticleDialogVisible=true"
          >新增技巧</el-button>
        </el-form-item>
      </el-form>
      <el-alert :title="alertText" type="info" class="alert" :closable="false" show-icon></el-alert>
      <!-- 文章列表区域 -->
      <el-table
        :data="articleDataList"
        style="width: 100%"
        :fit="true"
        highlight-current-row
        stripe
      >
        <el-table-column type="index" label="序号" width="80"></el-table-column>
        <el-table-column label="文章标题" width="400">
          <template slot-scope="scope">
            <span>{{scope.row.title}}</span>
            <el-link
              :underline="false"
              v-if="scope.row.videoURL"
              class="el-icon-film"
              @click="playVideoDialog(scope.row.videoURL)"
            ></el-link>
          </template>
        </el-table-column>
        <el-table-column prop="visits" label="阅读数"></el-table-column>
        <el-table-column prop="username" label="录入人" width="180"></el-table-column>
        <el-table-column label="录入时间">
          <template slot-scope="scope">
            <span>{{scope.row.createTime|parseTimeByString("{y}-{m}-{d} {h}:{i}:{s}")}}</span>
          </template>
        </el-table-column>
        <el-table-column label="状态" width="80">
          <template slot-scope="scope">
            <span>{{scope.row.state?'已启用':'已禁用'}}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="text" @click="previewArticleInfo(scope.row)">预览</el-button>
            <el-button
              type="text"
              @click="stateChangeOnclick(scope.row)"
            >{{scope.row.state?'禁用':'启用'}}</el-button>
            <el-button
              type="text"
              :disabled="scope.row.state?true:false"
              @click="showEditArticle(scope.row)"
            >修改</el-button>
            <el-button
              type="text"
              :disabled="scope.row.state?true:false"
              @click="removeArticle(scope.row)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <div class="pagination">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="articleListQueryInfo.page"
          :page-sizes="[3,5, 8, 10]"
          :page-size="articleListQueryInfo.pagesize"
          layout="prev, pager, next,sizes, jumper"
          :total="total"
        ></el-pagination>
      </div>
      <!-- 新增对话框，富文本编辑器 -->
      <el-dialog
        class="article-quill"
        title="提示"
        :visible.sync="addArticleDialogVisible"
        width="50%"
        @close="clearArticleInput"
      >
        <el-form ref="addArticleFormRef" :model="addArticleFormData" label-width="100px">
          <el-form-item label="文章标题：" required>
            <el-input v-model="addArticleFormData.title" placeholder="请输入文章标签"></el-input>
          </el-form-item>
          <el-form-item label="文章内容：" required>
            <quill-editor
              v-model="addArticleFormData.articleBody"
              ref="articleQtuillEditorRef"
              :options="editorOption"
            ></quill-editor>
          </el-form-item>
          <el-form-item label="视频地址：">
            <el-input placeholder="请输入视频地址" v-model="addArticleFormData.videoURL"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addArticleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addArticle">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑文章对话框 -->
      <el-dialog
        class="article-quill"
        title="提示"
        :visible.sync="editArticleDialogVisible"
        width="50%"
      >
        <el-form ref="editArticleFormRef" :model="editArticleFormData" label-width="100px">
          <el-form-item label="文章标题：" required>
            <el-input v-model="editArticleFormData.title" placeholder="请输入文章标签"></el-input>
          </el-form-item>
          <el-form-item label="文章内容：" required>
            <quill-editor
              v-model="editArticleFormData.articleBody"
              ref="articleQtuillEditorRef"
              :options="editorOption"
            ></quill-editor>
          </el-form-item>
          <el-form-item label="视频地址：">
            <el-input placeholder="请输入视频地址" v-model="editArticleFormData.videoURL"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editArticleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editArticle">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 播放视频 弹窗 -->
      <div v-bind:class="{videoActive:isActive,videoWrapper:isShow}" v-if="isActive">
        <div class="video-close-box">
          <div class="close-btn">
            <span class="el-icon-close" @click="closeVideoPlay"></span>
          </div>
          <div class="video-box">
            <video :src="currentVideoURL" muted autoplay width="640" height="368" controls loop></video>
          </div>
        </div>
      </div>
      <!-- 预览弹窗 -->
      <el-dialog
        title="预览窗口"
        :visible.sync="previewArticledialogVisible"
        width="800px"
        class="previewarticle-dialog"
      >
        <h2 class="article-title">{{previewArticleForm.title}}</h2>
        <div class="article-desc">
          <span>{{previewArticleForm.createTime|parseTimeByString("{y}-{m}-{d} {h}:{i}:{s}")}}</span>
          <span>{{previewArticleForm.createID}}</span>
          <i class="el-icon-view"></i>
          <span>{{previewArticleForm.visits}}</span>
        </div>
        <div class="article-content" v-html="previewArticleForm.articleBody"></div>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import { list, changeState, add, update, remove, detail } from '@/api/hmmm/articles.js'
import hljs from 'highlight.js'
export default {
  name: 'article-com',
  data () {
    return {
      alertText: '数据一共11条',
      articleListQueryInfo: {
        page: 1,
        pagesize: 10,
        keyword: '',
        state: null
      },
      total: null,
      articleDataList: [],
      addArticleDialogVisible: false,
      addArticleFormData: {
        title: '',
        articleBody: '',
        videoURL: ''
      },
      editorOption: {
        placeholder: '',
        modules: {
          toolbar: [
            ['bold', 'italic', 'underline', 'strike'], // 加粗，斜体，下划线，删除线
            [{ list: 'ordered' }, { list: 'bullet' }], // 有序列表，无序列表
            ['blockquote'], // 引用，代码块
            ['code-block', 'image', 'link']
          ],
          syntax: {
            highlight: (text) => hljs.highlightAuto(text).value
          }
        }
      },
      editArticleDialogVisible: false,
      editArticleFormData: {
        id: '',
        title: '',
        articleBody: '',
        videoURL: ''
      },
      currentVideoURL: null,
      isActive: false,
      isShow: true,
      previewArticledialogVisible: false,
      previewArticleForm: {}
    }
  },
  watch: {},
  computed: {},
  methods: {
    async loadArticleList () {
      try {
        this.articleListQueryInfo.keyword = this.articleListQueryInfo.keyword.trim()
        const { data } = await list(this.articleListQueryInfo)
        this.articleDataList = data.items
        this.total = data.counts
        this.alertText = `数据一共有${this.total}条`
      } catch (err) {
        this.$message.error('获取面试技巧失败！')
      }
    },
    handleSizeChange (val) {
      this.articleListQueryInfo.pagesize = val
      this.loadArticleList()
    },
    handleCurrentChange (val) {
      this.articleListQueryInfo.page = val
      this.loadArticleList()
    },
    // 搜索文章
    searchArticle () {
      this.loadArticleList()
    },
    clearInput () {
      this.articleListQueryInfo.keyword = ''
      this.articleListQueryInfo.state = null
      this.loadArticleList()
    },
    // 状态修改
    async stateChangeOnclick (row) {
      try {
        // const params = {
        //   id: row.id,
        //   state: row.state
        // }
        await changeState(row)
        const index = this.articleDataList.findIndex(i => i.id === row.id)
        this.articleDataList[index].state = !row.state
      } catch (err) { }
    },
    // 退出对话框清空编辑内容
    clearArticleInput () {
      // this.$refs.addArticleFormRef.resetFields()
      this.addArticleFormData.title = ''
      this.addArticleFormData.articleBody = ''
      this.addArticleFormData.videoURL = ''
    },
    // 新增或者添加文章
    async addArticle () {
      try {
        await add(this.addArticleFormData)
        this.loadArticleList()
        this.addArticleDialogVisible = false
      } catch (err) {

      }
    },
    // 显示修改文章对话框
    showEditArticle (row) {
      this.editArticleDialogVisible = true
      this.editArticleFormData.articleBody = row.articleBody
      this.editArticleFormData.id = row.id
      this.editArticleFormData.title = row.title
      this.editArticleFormData.videoURL = row.videoURL
    },
    // 修改文章
    async editArticle () {
      try {
        await update(this.editArticleFormData)
        this.$message.success('编辑文章成功')
        this.editArticleDialogVisible = false
        this.loadArticleList()
      } catch (err) { }
    },
    // 删除文章
    removeArticle (row) {
      try {
        remove(row).then(data => {
          this.$message('删除文章成功')
          this.loadArticleList()
        })
      } catch (err) {
        this.$message.error('删除失败')
      }
    },
    // 点击播放视频
    playVideoDialog (url) {
      this.currentVideoURL = url
      this.isActive = true
    },
    // 关闭视频播放
    closeVideoPlay () {
      this.isActive = false
    },
    // 预览文章详情
    previewArticleInfo (row) {
      this.previewArticledialogVisible = true
      try {
        detail(row).then(data => {
          this.previewArticleForm = data.data
        })
      } catch (err) {

      }
    }
  },
  created () {
    this.loadArticleList()
  },
  mounted () { }
}
</script>

<style scoped lang='scss'>
.filter-item {
  margin-left: 10px;
}
.el-alert {
  margin-bottom: 15px;
}
.pagination {
  margin-top: 18px;
  margin-bottom: 18px;
  float: right;
}
.quill-editor {
  min-height: 150px;
}
::v-deep .ql-editor {
  min-height: 150px;
}
// .ql-container {
//   min-height: 250px;
// }
.dialog-footer {
  display: flex;
  justify-content: center;
}
.el-icon-film {
  color: blue;
  margin-left: 10px;
  font-size: 18px;
}
// 视频盒子样式
.videoWrapper {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.3);
  overflow: hidden;
  z-index: 9999;
  .video-close-box {
    position: absolute;
    top: 50px;
    left: 50%;
    transform: translate(-50%);
    width: 900px;
    height: 600px;
    .close-btn {
      width: 50px;
      height: 50px;
      position: absolute;
      top: 30px;
      left: 50%;
      transform: translate(-50%, 0);
      background: rgba(0, 0, 0, 0.4);
      box-shadow: 0 0 5px rgb(0 0 0 / 40%);
      border-radius: 50%;
      text-align: center;
      line-height: 50px;
      color: #fff;
      font-size: 20px;
    }
    .video-box {
      position: absolute;
      left: 50%;
      top: 100px;
      width: 640px;
      height: 368px;
      transform: translate(-50%);
    }
  }
}
//预览详情样式
.article-content {
  background-color: #f5f5f5;
  padding: 10px;
  word-break: break-all;
  min-height: 40px;
}
.article-title {
  margin-top: 5px;
}

::v-deep .el-dialog__body {
  padding: 20px 18px;
}
.previewarticle-dialog {
  min-width: 730px !important;
}
.article-desc {
  min-width: 730px;
  margin-bottom: 10px;
  .el-icon-view {
    margin: 0 10px;
  }
}
</style>
