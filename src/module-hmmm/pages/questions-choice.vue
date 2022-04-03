<template>
  <div class="container paddingContainer">
    <el-card class="cardBody" shadow="always">
      <!-- 新增试题按钮 -->
      <el-button
        type="success"
        icon="el-icon-edit"
        size="small"
        class="addedBtn"
        @click="addTestQuestions"
      >新增试题</el-button>
      <!-- 表单区域 -->
      <el-form label-width="80px" :model="queryInfo" ref="formData">
        <el-row :gutter="20">
          <el-col :span="6">
            <el-form-item label="学科" prop="subjectID">
              <el-select
                placeholder="请选择"
                v-model="queryInfo.subjectID"
                @change="changeSubject(queryInfo.subjectID)"
              >
                <el-option
                  v-for="item in qusetionList"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="二级目录" prop="catalogID">
              <el-select placeholder="请选择" v-model="queryInfo.catalogID">
                <el-option
                  v-for="item in secondList"
                  :key="item.value"
                  :value="item.value"
                  :label="item.label"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="标签" prop="tags">
              <el-select placeholder="请选择" v-model="queryInfo.tags">
                <el-option
                  v-for="item in tagsList"
                  :key="item.value"
                  :value="item.value"
                  :label="item.label"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="关键字" prop="gjz">
              <el-input v-model="queryInfo.gjz"></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="6">
            <el-form-item label="试题类型" prop="questionType">
              <el-select placeholder="请选择" v-model="queryInfo.questionType">
                <el-option
                  v-for="item in questionTypeList"
                  :key="item.value"
                  :value="item.value"
                  :label="item.label"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="难度" prop="difficulty">
              <el-select placeholder="请选择" v-model="queryInfo.difficulty">
                <el-option
                  v-for="item in difficultyList"
                  :key="item.value"
                  :value="item.value"
                  :label="item.label"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="方向" prop="direction">
              <el-select placeholder="请选择" v-model="queryInfo.direction">
                <el-option
                  v-for="item in directionList"
                  :key="item.value"
                  :value="item"
                  :label="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="录入人" prop="creatorID">
              <el-select placeholder="请选择" v-model="queryInfo.creatorID">
                <el-option
                  v-for="item in creatorList"
                  :key="item.id"
                  :value="item.id"
                  :label="item.username"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="6">
            <el-form-item label="题目备注" prop="notes">
              <el-input v-model="queryInfo.notes"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="企业简称" prop="companyshort">
              <el-input v-model="queryInfo.companyshort"></el-input>
            </el-form-item>
          </el-col>
          <!-- 城市 -->
          <el-col :span="6">
            <el-form-item label="城市" prop="province">
              <el-select
                placeholder="请选择"
                style="width: 111px"
                v-model="queryInfo.province"
                @change="getcity()"
              >
                <el-option
                  v-for="item in this.provincesList"
                  :key="item.id"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
              <el-select
                placeholder="请选择"
                style="width: 111px"
                class="selectLocation"
                v-model="queryInfo.city"
              >
                <el-option
                  v-for="item in this.citysList"
                  :key="item.id"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item class="btnLocation">
              <el-button size="small" @click="resetForm">清除</el-button>
              <el-button type="primary" size="small" @click="search">搜索</el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <!-- tab栏切换 -->
      <el-tabs type="card" v-model="activeName" @tab-click="tabClick">
        <!-- 全部 -->
        <el-tab-pane label="全部" name="first"></el-tab-pane>
        <el-tab-pane label="待审核" name="second"></el-tab-pane>
        <el-tab-pane label="已审核" name="third"></el-tab-pane>
        <el-tab-pane label="已拒绝" name="fourth"></el-tab-pane>
        <!-- 总条数 -->
        <el-alert
          :title="'数据一共' + total + '条'"
          type="info"
          show-icon
          :closable="false"
          style="margin-bottom: 20px"
        ></el-alert>
        <!-- 表格区域 -->
        <el-table :data="dataList" style="width: 100%">
          <el-table-column prop="number" label="试题编号" width="120"></el-table-column>
          <el-table-column prop="subject" label="学科" width="120"></el-table-column>
          <el-table-column prop="catalog" label="目录" width="120"></el-table-column>
          <el-table-column prop="questionType" label="题型" width="120">
            <template slot-scope="scope">
              <span v-if="scope.row.questionType == 1">单选</span>
              <span v-else-if="scope.row.questionType == 2">多选</span>
              <span v-else>简单</span>
            </template>
          </el-table-column>
          <el-table-column prop="question" label="题干" width="280">
            <template slot-scope="scope">
              <div>{{ highlight(scope.row.question) }}</div>
            </template>
          </el-table-column>
          <el-table-column prop="addDate" label="录入时间" width="180">
            <template slot-scope="scope">{{ scope.row.addDate | parseTimeByString }}</template>
          </el-table-column>
          <el-table-column prop="difficulty" label="难度" width="80">
            <template slot-scope="scope">
              <span v-if="scope.row.difficulty == 1">简单</span>
              <span v-else-if="scope.row.difficulty == 2">一般</span>
              <span v-else>困难</span>
            </template>
          </el-table-column>
          <el-table-column prop="creator" label="录入人" width="120"></el-table-column>
          <!-- 审核状态 -->
          <el-table-column prop="chkState" label="审核状态" width="120">
            <template slot-scope="scope">
              <span v-if="scope.row.chkState==1">待审核</span>
              <span v-else-if="scope.row.chkState==2">已通过</span>
              <span v-else>已拒绝</span>
            </template>
          </el-table-column>
          <!-- 审核意见 -->
          <el-table-column prop="chkRemarks" label="审核意见" width="150"></el-table-column>
          <!-- 审核人 -->
          <el-table-column prop="chkUser" label="审核人" width="120"></el-table-column>
          <!-- 发布状态 -->
          <el-table-column prop="publishState" label="发布状态" width="150">
            <template slot-scope="scope">
              <span v-if="scope.row.publishState==1">待发布</span>
              <span v-else-if="scope.row.publishState==2">已发布</span>
              <span v-else>已下架</span>
            </template>
          </el-table-column>
          <!-- 操作 -->
          <el-table-column fixed="right" label="操作" width="200">
            <template slot-scope="scope">
              <!-- 预览 -->
              <el-button @click="quesPreview(scope.row)" type="text" size="small">预览</el-button>
              <!-- 审核 -->
              <el-button
                @click="audit(scope.row)"
                :disabled="scope.row.chkState == 1 ? true : false"
                type="text"
                size="small"
              >审核</el-button>
              <!-- 修改 -->
              <el-button type="text" size="small" @click="xiugai(scope.row)">修改</el-button>
              <!-- 下架 -->
              <el-button
                type="text"
                size="small"
                @click="shelves(scope.row)"
                :disabled="idUpDown(scope.row)"
              >{{scope.row.publishState == 1 ? "下架" : "上架"}}</el-button>
              <el-button type="text" size="small" @click="del(scope.row.id)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-tabs>

      <!-- 分页 -->

      <el-pagination
        class="paginationLoation"
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryData.page"
        :page-sizes="[2,3,4,7]"
        :page-size="queryData.pagesize"
        layout="sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 预览dialog对话框 -->
    <el-dialog title="题目预览" :visible.sync="questionsDialogVisible" width="50%">
      <el-form :model="questionForm" label-width="80px">
        <el-row :gutter="20">
          <el-col :span="6">
            <div>
              【题型】:
              <span v-if="questionForm.questionType==1">单选</span>
              <span v-else-if="questionForm.questionType==2">多选</span>
              <span v-else>简答</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div>
              【编号】:
              <span>{{questionForm.id}}</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div>
              【难度】:
              <span v-if="questionForm.difficulty==1">简单</span>
              <span v-else-if="questionForm.difficulty==2">一般</span>
              <span v-else>困难</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div>
              【标签】:
              <span>{{questionForm.tags}}</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div>
              【学科】:
              <span>{{questionForm.subject}}</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div>
              【目录】:
              <span>{{questionForm.catalog}}</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div>
              【方向】:
              <span>{{questionForm.direction}}</span>
            </div>
          </el-col>
        </el-row>
        <hr>
        <div>
          【题干】:
          <div v-html="questionForm.question"></div>
        </div>

        <!-- <div>
  <span v-if="questionForm.questionType==1">单选</span>
  <span v-else-if="questionForm.questionType==2">多选</span>
  <div v-else>简答</div>题 选项：(以下选中的选项为正确答案)<br />

 <el-radio
    class="radioLocation"
    v-model="radio"
    label="1"
    v-for="item in options"
    :key="item.id">

    {{item.title}}
    </el-radio>

   <br/>
        </div>-->
        <div
          class="hint"
        >{{['单选题 选项：（以下选中的选项为正确答案）','多选题 选项：（以下选中的选项为正确答案）','简答题：'][questionForm.questionType - 0]}}</div>
        <div v-if="questionForm.questionType==1">
          <div class="radio-item" v-for="(option,i) in options" :key="i">
            <el-radio :label="option.title" :value="option.isRight?option.title:''"></el-radio>
          </div>
        </div>
        <div v-if="questionForm.questionType==2">
          <div v-for="(option,i) in options" :key="i">
            <el-checkbox :label="option.title" :checked="option.isRight==1?true:false"></el-checkbox>
          </div>
        </div>
        <hr>
        <div>
          【参考答案】:
          <el-button type="danger" @click="showVideo=true">视频答案预览</el-button>
          <div v-if="showVideo" class="video">
            <video :src="questionForm.videoURL" autoplay controls loop></video>
          </div>
        </div>
        <hr>
        <div>
          【答案解析】:
          <span v-html="questionForm.answer"></span>
        </div>
        <hr>
        <div>
          【题目备注】:
          <span>{{questionForm.remarks}}</span>
        </div>
      </el-form>
      <span slot="footer">
        <el-button type="primary" @click="handleClose" class="closetBtnLocation">关 闭</el-button>
      </span>
    </el-dialog>

    <!-- 审核对话框 -->
    <el-dialog title="题目审核" :visible.sync="auditDialogVisible" width="30%" @close="auditClose">
      <el-form ref="auditFormRef" :model="auditList" :rules="auditRules">
        <el-form-item prop="chkState">
          <el-radio v-model="auditList.chkState" label="1">通过</el-radio>
          <el-radio v-model="auditList.chkState" label="2">拒绝</el-radio>
        </el-form-item>
        <el-form-item prop="chkRemarks">
          <el-input type="textarea" :rows="2" placeholder="请输入审核意见" v-model="auditList.chkRemarks"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="auditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="onaudit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { simple as ssimple } from '@/api/hmmm/subjects'
import { simple as usimple } from '@/api/base/users'
import { simple as dsimple } from '@/api/hmmm/directorys'
import { simple as tsimple } from '@/api/hmmm/tags'
import { remove as delquestionById, detail as qdetail, choice as qchoice, choicePublish, choiceCheck } from '@/api/hmmm/questions'// 获取基础题库
import { provinces, citys } from '@/api/hmmm/citys'
export default {
  data () {
    return {
      // 基础题库列表信息
      queryInfo: {
        subjectID: '',
        catalogID: '',
        questionType: '',
        question: '',
        difficulty: '',
        creatorID: '',
        province: '',
        tags: '',
        direction: '',
        chkRemarks: '',
        chkUser: ''

      },
      queryData: {
        page: 1,
        pagesize: 5

      },
      activeName: 'first',
      // 单选按钮绑定值
      radio: '',
      // 存放列表详情的数据
      options: [],
      // 控制预览弹出框的显示隐藏
      questionsDialogVisible: false,
      // 预览数据
      questionForm: [],
      // 视频框显示隐藏
      showVideo: false,
      // 控制审核弹出框显示隐藏
      auditDialogVisible: false,
      // 审核数据
      auditList: { id: '', chkState: '', chkRemarks: '' },
      // 审核规则
      auditRules: {
        chkState: [{ required: true, message: '请选择是否通过', trigger: 'blur' }],
        chkRemarks: [{ required: true, message: '请输入审核意见', trigger: 'blur' }]
      },
      // 总条数
      total: 0,
      // 学科列表
      qusetionList: [],
      // 二级目录
      secondList: [],
      // 标签列表
      tagsList: [],
      // 录入人
      creatorList: [],
      // 精选题库列表
      dataList: [],
      // 试题类型
      questionTypeList: [
        {
          value: 1,
          label: '单选'
        },
        {
          value: 2,
          label: '多选'
        },
        {
          value: 3,
          label: '简答'
        }
      ],
      // 难度
      difficultyList: [
        {
          value: 1,
          label: '简单'
        },
        {
          value: 2,
          label: '一般'
        },
        {
          value: 3,
          label: '困难'
        }
      ],
      // 方向
      directionList: [
        'o2o',
        '外包服务',
        '企业服务',
        '互联网金融',
        '企业咨询',
        '互联网',
        '电子商务',
        '其他'
      ],

      provincesList: provinces(),
      citysList: [],
      // 审核状态
      chkTypeList: [
        {
          value: 1,
          label: '待审核'
        },
        {
          value: 2,
          label: '通过'
        },
        {
          value: 3,
          label: '拒绝'
        }
      ],
      // 发布状态
      publishTypeList: [
        {
          value: 1,
          label: '待发布'
        },
        {
          value: 2,
          label: '已发布'
        },
        {
          value: 3,
          label: '已下架'
        }
      ]
    }
  },
  // 生命周期函数
  created () {
    // 获取学科名称
    this.getQuestionsData()
    this.getDataList()
  },

  methods: {
    // 点击跳转到试题录入页面
    addTestQuestions () {
      this.$router.push('/questions/new')
    },
    // 点击修改跳转到试题录入
    xiugai (row) {
      this.$router.push({ path: '/questions/new', query: { id: row.id } })
    },
    // 获取学科列表
    async getQuestionsData () {
      try {
        const { data: res1 } = await ssimple()
        this.qusetionList = res1
        console.log(this.qusetionList)
      } catch (err) {
        console.log(err)
      }
      // 获取录入人
      const { data: res2 } = await usimple()
      this.creatorList = res2
    },
    // 变化
    async changeSubject (id) {
      // console.log(id)
      const { data: res3 } = await dsimple({ subjectID: id })
      this.secondList = res3
      // console.log(res3)
      const { data: res4 } = await tsimple({ subjectID: id })
      this.tagsList = res4
    },
    // 获取精选题库列表
    async getDataList () {
      const { data: res5 } = await qchoice(this.queryData)
      this.dataList = res5.items
      this.total = res5.counts
      console.log(this.dataList)
    },
    // 搜索
    async search () {
      const { data: res6 } = await qchoice(this.queryData)
      this.dataList = res6.items
      console.log(this.dataList)
    },
    // 预览事件
    async quesPreview (row) {
      this.questionsDialogVisible = true
      this.questionForm = row
      const { data: qinfo } = await qdetail(row)
      this.options = qinfo.options
      console.log(this.options)
    },
    // 分页变化事件
    // 只要pagesize发生了变化就会触发这个函数
    handleSizeChange (newSize) {
      this.queryData.pagesize = newSize
      this.getDataList()
    },
    // 只要pagesize发生了变化就会触发这个函数
    async handleCurrentChange (newPage) {
      this.queryData.page = newPage
      this.getDataList()
    },
    // 去除表格中所有html标签
    highlight (item) {
      return item.replace(/<[^>]+>/g, '')
    },

    // 删除
    del (id) {
      this.$confirm('此操作将永久删除该题目, 是否继续? ', '提示', {
        distinguishCancelAndClose: true,
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async _ => {
          await delquestionById({ id: id })
          this.$message.success('删除成功')
          if (this.dataList.length === 1 && this.queryData.page > 1) {
            this.queryData.page -= 1
          }
          this.getDataList()
        })
        .catch(() => {
          this.$message.info('已取消删除')
          this.getDataList()
        })
    },
    // 编辑跳转
    editBtn () {
      this.$router.push('/questions/new')
    },

    // 获取城市数据
    getcity () {
      this.citysList = citys(this.queryInfo.province)
    },

    resetForm () {
      // this.queryInfo.subjectID = null
      // this.queryInfo.catalogID = null
      // this.queryInfo.tags = null
      // this.queryInfo.gjz = null
      // this.queryInfo.questionType = null
      // this.queryInfo.difficulty = null
      // this.queryInfo.direction = null
      // this.queryInfo.creatorID = null
      // this.queryInfo.notes = null
      // this.queryInfo.companyshort = null
      // this.queryInfo.province = null

      this.$refs.formData.resetFields()
      this.queryInfo.city = null
    },
    // 预览框视频框显示与隐藏
    handleClose () {
      this.questionsDialogVisible = false
      this.showVideo = false
    },

    // tab栏点击事件
    async tabClick () {
      if (this.activeName === 'first') {
        const { data: res } = await qchoice()
        // console.log(res);
        this.counts = res.counts
        this.tableData = res.items
        this.getDataList()
      }
      if (this.activeName === 'second') {
        const { data: res } = await qchoice({ chkState: 1 })
        this.total = res.counts
        this.dataList = res.items
        // console.log(this.dataList)
      }
      if (this.activeName === 'third') {
        const { data: res } = await qchoice({ chkState: 2 })
        this.total = res.counts
        this.dataList = res.items
        // console.log(this.dataList)
      }
      if (this.activeName === 'fourth') {
        const { data: res } = await qchoice({ chkState: 3 })
        this.total = res.counts
        this.dataList = res.items
        // console.log(this.dataList)
      }
    },
    // 审核
    async audit (row) {
      console.log(row)
      this.auditList.id = row.id
      this.auditDialogVisible = true
    },
    // 发送审核请求
    onaudit () {
      this.$refs.auditFormRef.validate(async (valid) => {
        if (!valid) {
          return console.log(11)
        }
        try {
          console.log(this.auditList.chkState)
          this.auditList.chkState = parseInt(this.auditList.chkState)
          await choiceCheck(this.auditList)
          this.$message.success('操作成功！')
          this.getDataList()
          this.auditDialogVisible = false
        } catch (err) {
          this.$message.error('操作失败！')
        }
      })
    },
    // 关闭对话框清空数据
    auditClose () {
      this.$refs.auditFormRef.resetFields()
    },
    // 下架
    shelves (row) {
      this.$confirm('您确认下架这道题目吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          // 当前下架，进行上架
          if (row.publishState === 0) {
            const date = {
              id: row.id,
              publishState: 1
            }
            this.onUp(date)
            row.publishState = 1
          } else {
            // 当前上架，进行下架
            const date = {
              id: row.id,
              publishState: 0
            }
            this.onUp(date)
            row.publishState = 0
          }
        })
        .catch(() => {
          this.$message.info('已取消下架')
        })
    },
    // 上下架请求处理
    async onup (date) {
      try {
        await choicePublish(date)
        this.$message.success('操作成功')
      } catch (err) {
        this.$message.error('操作失败！')
      }
    }
  },
  computed: {
    idUpDown () {
      return function (date) {
        if (date.chkState === 1) {
          return false
        } else {
          return true
        }
      }
    }
  }
}
</script>

<style scoped lang='scss'>
.cardBody {
  .addedBtn {
    margin-left: 1175px;
    margin-bottom: 20px;
  }
  .selectLocation {
    padding-left: 4px;
  }
  .btnLocation {
    margin-left: 100px;
  }
  .el-input {
    width: 222px;
    height: 43px;
  }
  .paginationLoation {
    float: right;
    margin: 15px 0;
  }
}
.radioLocation {
  display: flex;
  // flex-direction: column;
  margin-top: 15px;
}
.video {
  width: 400px;
  height: 300px;
  video {
    width: 100%;
    height: 100%;
  }
}

.closetBtnLocation {
  display: block;
  margin: 0 auto;
}
</style>
