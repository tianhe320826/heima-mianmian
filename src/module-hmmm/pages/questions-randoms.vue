<template>
  <div class="randoms-container paddingContainer">
    <el-card>
      <!-- 关键字搜索 -->
      <el-form label-width="80px" :model="randomsQueryInfo">
        <el-form-item label="关键字">
          <el-input style="width: 200px" v-model="randomsQueryInfo.keyword" @clear="getRandomsList"></el-input>
          <div class="randomsBtn">
            <el-button size="small" @click="clearFormValue">清除</el-button>
            <el-button size="small" type="primary" @click="handleKeyWord">搜索</el-button>
          </div>
        </el-form-item>
        <el-alert :title="alertText" type="info" :closable="false" class="randomsAlert" show-icon></el-alert>
      </el-form>

      <!-- 组题列表 -->
      <el-table :data="quesRandomList" fit highlight-current-row style="width: 100%">
        <el-table-column prop="id" label="编号" width="220"></el-table-column>
        <el-table-column label="题型" prop="questionType" width="80">
          <template slot-scope="scope">
            <span v-if="scope.row.questionType === '1'">单选</span>
            <span v-if="scope.row.questionType === '2'">多选</span>
            <span v-if="scope.row.questionType === '3'">简答</span>
          </template>
        </el-table-column>
        <el-table-column prop="questionIDs" label="题目编号" width="220">
          <template slot-scope="scope">
            <el-link
              type="primary"
              v-for="(item, index) in scope.row.questionIDs"
              :key="index"
              @click="clickDialog(item.id)"
            >{{ item.number }}</el-link>
          </template>
        </el-table-column>
        <el-table-column prop="addTime" label="录入时间" width="180"></el-table-column>
        <el-table-column prop="totalSeconds" label="答题时间(s)"></el-table-column>
        <el-table-column prop="accuracyRate" label="正确率(%)"></el-table-column>
        <el-table-column prop="userName" label="录入人"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="danger"
              plain
              size="mini"
              icon="el-icon-delete"
              circle
              @click="deleteQuestions(scope.row)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页列表 -->
      <el-pagination
        style="margin-top: 18px"
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="randomsQueryInfo.page"
        layout="prev, pager, next, sizes,jumper"
        :page-sizes="[20, 30, 50, 100]"
        :page-size="randomsQueryInfo.pagesize"
        :total="total"
        align="right"
      ></el-pagination>
    </el-card>

    <!-- 题目预览对话框 -->
    <div class="dialogPreview">
      <el-dialog title="题目预览" :visible.sync="showQuestionPreview" width="60%" center>
        <el-row class="firstRow">
          <el-col prop="questionType" :span="6">
            【题型】：
            <span v-if="questionPreviewData.questionType==='1'">单选</span>
            <span v-if="questionPreviewData.questionType==='2'">多选</span>
            <span v-if="questionPreviewData.questionType==='3'">简答</span>
          </el-col>
          <el-col prop="number" :span="6">【编号】：{{questionPreviewData.id}}</el-col>
          <el-col prop="difficulty" :span="6">
            【难度】：
            <span>{{questionPreviewData.difficulty}}</span>
          </el-col>
          <el-col prop="tags" :span="6">
            【标签】：
            <span>{{questionPreviewData.tags}}</span>
          </el-col>
          <el-col prop="subjectID" :span="6">
            【学科】：
            <span>{{questionPreviewData.subjectName}}</span>
          </el-col>
          <el-col prop="catalogID" :span="6">
            【目录】：
            <span>{{questionPreviewData.directoryName}}</span>
          </el-col>
          <el-col prop="direction" :span="6">
            【方向】：
            <span>{{questionPreviewData.direction}}</span>
          </el-col>
        </el-row>
        <hr>
        <el-row>
          <el-col prop="question">
            【题干】：
            <div v-html="questionPreviewData.question" class="question-title-blue"></div>
          </el-col>
          <el-col style="padding-bottom: 5px">
            <span v-if="questionPreviewData.questionType==='1'">单选题</span>
            <span v-if="questionPreviewData.questionType==='2'">多选题</span>选项: (以下选中的选项为正确答案)
            <div v-if="questionPreviewData.questionType==='1'">
              <el-radio
                :label="1"
                :value="item.isRight"
                v-for="(item,index) in questionPreviewData.options"
                :key="index"
              >{{item.title}}</el-radio>
            </div>
            <div v-if="questionPreviewData.questionType==='2'">
              <el-radio
                :label="1"
                :value="item.isRight"
                v-for="(item,index) in questionPreviewData.options"
                :key="index"
              >{{item.title}}</el-radio>
            </div>
          </el-col>
        </el-row>
        <hr>
        <el-row>
          <el-col :span="4" style="padding-top: 8px">【参考答案】：</el-col>
          <el-button @click="playVideoAnswer" class="previewButton" type="danger" size="mini">视频答案预览</el-button>
          <video
            :src="questionPreviewData.videoURL"
            muted
            autoplay
            controls
            v-bind:class="[showOrnot,'video-none']"
          ></video>
        </el-row>
        <hr>
        <el-row>
          <el-col prop="answer" style="padding:  0">
            【答案解析】：
            <span v-html="questionPreviewData.answer"></span>
          </el-col>
        </el-row>
        <hr>
        <el-row>
          <el-col prop="remarks" style="padding: 8px 0">
            【题目备注】：
            <span>{{questionPreviewData.remarks}}</span>
          </el-col>
          <el-col></el-col>
        </el-row>
        <span slot="footer" class="dialog-footer">
          <el-button type="primary" @click="closeDialogAndVideo">关闭</el-button>
        </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { randoms, removeRandoms, detail } from '@/api/hmmm/questions.js'
export default {
  name: 'questionsRandoms',
  data () {
    return {
      alertText: '',
      // table数据
      quesRandomList: [],
      // 分页列表
      randomsQueryInfo: {
        page: 1,
        pagesize: 20,
        keyword: ''
      },
      total: 0,
      counts: 0,
      // 基础题库列表数据
      baseQuestionsList: [],
      // 题型展示预览
      showQuestionPreview: false,
      // 展示视频
      innerPreviewVideo: false,
      // 题目预览数据
      questionPreviewData: {},
      showOrnot: ''
    }
  },
  created () {
    this.getRandomsList()
  },
  methods: {
    // 获取table数据列表
    async getRandomsList () {
      try {
        const { data: res } = await randoms(this.randomsQueryInfo)
        // console.log(res);
        this.quesRandomList = res.items
        this.alertText = `数据一共 ${res.counts} 条`
        this.total = res.counts
      } catch (err) {
        this.$message.error('获取题组列表失败')
      }
    },
    // 获取基础题库数据
    async getBaseQuestionsList () {
      try {
        const { data: res } = await detail(this)
        console.log(res)
        this.baseQuestionsList = res.items
      } catch (err) {
        this.$message.error('获取题目数据失败！')
      }
    },
    // 控制每页显示数据
    handleSizeChange (newSize) {
      this.randomsQueryInfo.pagesize = newSize
      this.getRandomsList()
    },
    // 显示第几页数据
    handleCurrentChange (newPage) {
      this.randomsQueryInfo.page = newPage
      this.getRandomsList()
    },
    // 清除关键字搜索框
    clearFormValue () {
      this.randomsQueryInfo.keyword = ''
      this.getRandomsList()
    },
    // 点击搜索关键字
    handleKeyWord () {
      this.randomsQueryInfo.keyword = this.randomsQueryInfo.keyword.trim()
      this.getRandomsList()
    },
    // 删除题组
    deleteQuestions (row) {
      this.$confirm('此操作将永久删除该题组, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          removeRandoms(row).then((res) => {
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
            console.log(row)
            this.getRandomsList()
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 显示对话框
    clickDialog (id) {
      this.showQuestionPreview = true
      // this.getBaseQuestionsList();
      try {
        detail({ id: id }).then(data => {
          console.log(data)
          this.questionPreviewData = data.data
        })
      } catch (err) {
        console.log(err)
      }
    },
    // 显示预览视频
    clickPreviewVideo () {
      this.innerPreviewVideo = true
    },
    playVideoAnswer () {
      this.showOrnot = 'showIt'
    },
    closeDialogAndVideo () {
      this.showOrnot = ''
      this.showQuestionPreview = false
    }
  }
}
</script>

<style scoped lang='scss'>
.randomsBtn {
  float: right;
  margin-left: 10px;
}
.randomsAlert {
  margin-bottom: 14px;
}
.el-table-column {
  padding: 10px 0 !important;
}
.firstRow {
  margin-top: 10px;
}
.firstRow > .el-col {
  padding: 10px 0 !important;
}
.previewButton {
  padding: 9px 15px;
}
.question-title-blue {
  color: blue;
}
.el-radio {
  display: block;
  margin: 15px 0;
}
.video-none {
  display: none;
}
.showIt {
  display: block;
}
video {
  max-width: 400px;
  min-height: 300px;
}
</style>
