<template>
  <div class="container paddingContainer">
    <el-card>
      <div slot="header">{{ $route.query.id ? "试题修改" : "试题录入" }}</div>

      <el-form :model="Form" :rules="FormRules" ref="FormRef" label-width="100px">
        <el-form-item label="学科：" prop="subjectID">
          <el-select v-model="Form.subjectID" placeholder="请选择" @change="subjectChange">
            <el-option
              v-for="(item, index) in subjectList"
              :key="index"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="目录：" prop="catalogID">
          <el-select v-model="Form.catalogID" placeholder="请选择">
            <el-option
              v-for="(item, index) in catalogList"
              :key="index"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="企业：" prop="enterpriseID">
          <el-select v-model="Form.enterpriseID" placeholder="请选择">
            <el-option
              v-for="item in companyList"
              :key="item.id"
              :label="item.company"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-row>
          <el-col :span="3">
            <el-form-item label="城市：" prop="province">
              <el-select
                v-model="Form.province"
                placeholder="请选择"
                @change="provinceChange"
                class="province-select"
              >
                <el-option
                  v-for="(item, index) in provincesList"
                  :key="index"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="3">
            <el-form-item prop="city">
              <el-select v-model="Form.city" placeholder="请选择" class="province-select">
                <el-option
                  v-for="(item, index) in citysList"
                  :key="index"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item label="方向：" prop="direction">
          <el-select v-model="Form.direction" placeholder="请选择">
            <el-option
              v-for="(item, index) in directionList"
              :key="index"
              :label="item"
              :value="item"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="题型：" prop="questionType">
          <el-radio-group v-model="Form.questionType" @change="questionTypeChange">
            <el-radio :label="1">单选</el-radio>
            <el-radio :label="2">多选</el-radio>
            <el-radio :label="3">简答</el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="难度：" prop="difficulty">
          <el-radio-group v-model="Form.difficulty" @change="difficultyChange">
            <el-radio :label="1">简单</el-radio>
            <el-radio :label="2">一般</el-radio>
            <el-radio :label="3">困难</el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="题干：" prop="question" class="question-form-item">
          <quill-editor v-model="Form.question" :options="editorOption"></quill-editor>
        </el-form-item>

        <el-form-item label="选项：" v-if="Form.questionType != 3">
          <div class="options-box" v-for="(item, index) in Form.options" :key="index">
            <el-radio
              v-if="Form.questionType == 1"
              :label="true"
              v-model="item.isRight"
              @change="radioChange(index)"
            >{{ item.code + "：" }}</el-radio>
            <el-checkbox v-else v-model="item.isRight">
              {{
              item.code + "："
              }}
            </el-checkbox>
            <el-input v-model="item.title"></el-input>
            <el-upload
              class="avatar-uploader"
              action="https://jsonplaceholder.typicode.com/posts/"
              :show-file-list="false"
              :on-success="handleAvatarSuccess.bind(null, index)"
              :before-upload="beforeAvatarUpload"
            >
              <img v-if="item.img" :src="item.img" class="avatar">
              <span v-else class="tetx">上传图片</span>
              <i class="el-icon-circle-close avatar-uploader-icon" @click="delPhoto($event, index)"></i>
            </el-upload>
          </div>
          <el-button
            type="danger"
            :disabled="Form.questionType != 2"
            size="small"
            class="add-btn"
            @click="addOptions"
          >+增加选项与答案</el-button>
        </el-form-item>

        <el-form-item label="解析视频：">
          <el-input v-model="Form.videoURL" class="video-input"></el-input>
        </el-form-item>

        <el-form-item label="答案解析：" prop="answer" class="question-form-item">
          <quill-editor v-model="Form.answer" :options="editorOption"></quill-editor>
        </el-form-item>

        <el-form-item label="题目备注：" class="textarea-form-item">
          <el-input type="textarea" :rows="4" v-model="Form.remarks"></el-input>
        </el-form-item>

        <el-form-item label="试题标签：" class="textarea-form-item">
          <el-select v-model="Form.tags" multiple placeholder="请选择试题标签">
            <el-option
              v-for="item in tagsList"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item>
          <el-button
            v-if="!$route.query.id"
            type="primary"
            size="medium"
            @click="submitForm('FormRef')"
          >确认提交</el-button>
          <el-button v-else type="success" size="medium" @click="alter('FormRef')">确认修改</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import { provinces, citys } from '@/api/hmmm/citys'
import { simple as getSubjects } from '@/api/hmmm/subjects'
import { simple as getDirectorys } from '@/api/hmmm/directorys'
import { list as getCompany } from '@/api/hmmm/companys'
import { direction } from '@/api/hmmm/constants'
import { simple as getTags } from '@/api/hmmm/tags'
import { add, detail, update } from '@/api/hmmm/questions'
// 局部引用富文本编辑器
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'

import { quillEditor } from 'vue-quill-editor'
// 引用代码高亮插件
import hljs from 'highlight.js'
import 'highlight.js/styles/monokai-sublime.css' // 代码高亮样式

export default {
  components: { quillEditor },
  data () {
    return {
      // 表单的数据
      Form: {
        subjectID: '',
        catalogID: '',
        enterpriseID: '',
        province: '',
        city: '',
        direction: '',
        questionType: 1,
        difficulty: 1,
        question: '',
        options: [
          {
            code: 'A',
            title: '',
            img: null,
            isRight: true
          },
          {
            code: 'B',
            title: '',
            img: null,
            isRight: false
          },
          {
            code: 'C',
            title: '',
            img: null,
            isRight: false
          },
          {
            code: 'D',
            title: '',
            img: null,
            isRight: false
          }
        ],
        videoURL: null,
        answer: null,
        remarks: null,
        tags: null
      },
      // 表单的验证规则
      FormRules: {
        subjectID: [
          { required: true, message: '请选择学科', trigger: 'change' }
        ],
        catalogID: [
          { required: true, message: '请选择目录', trigger: 'change' }
        ],
        enterpriseID: [
          { required: true, message: '请选择企业', trigger: 'change' }
        ],
        province: [
          { required: true, message: '请选择城市', trigger: 'change' }
        ],
        city: [{ required: true, message: '请选择地区', trigger: 'change' }],
        direction: [
          { required: true, message: '请选择方向', trigger: 'change' }
        ],
        question: [{ required: true, message: '请输入题干', trigger: 'blur' }],
        answer: [
          { required: true, message: '请输入答案解析', trigger: 'blur' }
        ],
        questionType: [
          { required: true, message: '请选择题型', trigger: 'blur' }
        ],
        difficulty: [
          { required: true, message: '请选择难度', trigger: 'blur' }
        ]
      },
      // 学科的列表
      subjectList: [],
      // 目录的列表
      catalogList: [],
      // 企业的列表
      companyList: [],
      // 引入城市与地区
      provincesList: provinces(),
      citysList: [],
      // 引入方向的列表
      directionList: direction,
      // 试题标签的列表
      tagsList: [],
      // 富文本编辑器的配置
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
      }
    }
  },
  methods: {
    async geiSubjectList () {
      const { data: res } = await getSubjects()
      this.subjectList = res
    },
    // 选择学科后，更新目录选项 与 更新标签选项
    async subjectChange () {
      // 更新目录列表 重置目录下拉框
      const { data: res } = await getDirectorys(this.Form)
      this.catalogList = res
      this.Form.catalogID = ''
      // 更新标签列表 重置标签下拉框
      const { data: res1 } = await getTags(this.Form)
      this.tagsList = res1
      this.Form.tags = null
    },
    async getcompanysList () {
      const { data: res } = await getCompany()
      this.companyList = res.items
    },
    provinceChange (val) {
      this.citysList = citys(val)
    },
    questionTypeChange (val) {
      // 改变题型后 初始化配置数组
      if (val === 1) {
        // 1.单选 将多余的数组项删除 重置选项A
        this.Form.options.splice(4)
        this.Form.options.forEach((ele) => {
          ele.isRight = false
        })
        this.Form.options[0].isRight = true
      }
      // 2.多选 可增加数组成员 在add-btn按钮的点击事件中
      // 3.简答 数组重置 在提交的时候才处理 增加用户体验
    },
    difficultyChange (val) {
      // 改变难度后 改变数据
      this.Form.difficulty = val
    },
    radioChange (index) {
      // 点击单选框 改变正确答案的值
      this.Form.options.forEach((ele, i) => {
        i !== index ? (ele.isRight = false) : (ele.isRight = true)
      })
    },
    // 判断上传图片格式是否正确
    beforeAvatarUpload (file) {
      const isJPG = file.type === 'image/jpeg'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isJPG && isLt2M
    },
    // 上传图片后 替换预览图
    handleAvatarSuccess (index, res, file) {
      this.Form.options[index].img = URL.createObjectURL(file.raw)
    },
    delPhoto (event, index) {
      event.stopPropagation()
      this.Form.options[index].img = null
    },
    // 增加选项按钮 向数组里添加成员
    addOptions () {
      const code = String.fromCharCode(this.Form.options.length + 65)
      const obj = { code, title: '', img: null, isRight: false }
      this.Form.options.push(obj)
    },
    // 提交表单事件 注意数据里数字需改成字符串 注意如题型为简答，需重置Form.options
    submitForm (FormRef) {
      this.$refs[FormRef].validate(async (valid) => {
        // 表单校验
        if (!valid) {
          return this.$message.error('请填写完整再提交！')
        } else {
          // 校验通过 提交表单
          const copyForm = this.dataProcess()
          await add(copyForm)
          this.$message.success('添加成功！')
          this.$router.push('/questions/list')
        }
      })
    },
    // 提交修改后的表单
    alter (FormRef) {
      this.$refs[FormRef].validate(async (valid) => {
        if (!valid) {
          return this.$message.error('请填写完整再提交！')
        } else {
          const copyForm = this.dataProcess()
          await update(copyForm)
          this.$message.success('修改成功！')
          this.$router.push('/questions/list')
        }
      })
    },
    // 处理提交数据的函数
    dataProcess () {
      // 深拷贝一个对象 用拷贝的对象提交
      const copyForm = JSON.parse(JSON.stringify(this.Form))
      // 判断 题型是简答 清空选择的数组
      copyForm.questionType === 3 && (copyForm.options = [])
      // 将个别数据格式处理
      copyForm.questionType += ''
      copyForm.difficulty += ''
      copyForm.tags = JSON.stringify(copyForm.tags)
      return copyForm
    },
    // 如果this.$route.query.id存在 请求参数 渲染页面
    async renderPage () {
      if (this.$route.query.id) {
        const { data: res } = await detail(this.$route.query)
        res.questionType = res.questionType / 1
        res.difficulty = res.difficulty / 1
        res.tags = res.tags.split(',')
        res.options.forEach((ele) => {
          ele.isRight = Boolean(ele.isRight)
        })
        res.options.sort((a, b) => {
          return a.code.charCodeAt() - b.code.charCodeAt()
        })
        this.Form = res
      }
    }
  },
  mounted () {
    this.geiSubjectList()
    this.getcompanysList()
    this.renderPage()
  },
  watch: {
    $route () {
      if (!this.$route.query.id) {
        // 重置表单
        this.$refs.FormRef.resetFields()
        this.Form.options.forEach((ele) => {
          ele.title = ''
        })
        this.Form.tags = []
        this.Form.videoURL = ''
        this.Form.remarks = ''
      }
    }
  }
}
</script>

<style scoped lang='scss'>
.el-select {
  width: 400px;
}
.province-select {
  width: 198px;
  margin-right: -10px;
}
.el-col-3 {
  width: 12.2%;
  min-width: 200px;
}
.el-form {
  margin-left: 20px;
}
.question-form-item {
  height: 239px;
}
.add-btn {
  margin-top: 5px;
}
.video-input {
  width: 400px;
}

.textarea-form-item {
  width: 500px;
}
</style>
