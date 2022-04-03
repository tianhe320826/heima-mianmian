
  <!-- <div class='quePreview'>
      <el-dialog
        title="题目预览"
        :visible.sync="dialogTableVisible"
        width="60%"
        :before-close="handleClose"
        >

 <el-row>
   <el-col :span="6">【题型】:{{questionType}}</el-col>
   <el-col :span="6">【编号】:{{prevList.id}}</el-col>
   <el-col :span="6">【难度】:{{difficult}}</el-col>
   <el-col :span="6">【标签】:{{prevList.tags}}</el-col>
   <el-col :span="6">【学科】:{{subject}}</el-col>
   <el-col :span="6">【目录】:{{directory}}</el-col>
   <el-col :span="6">【方向】:{{prevList.direction}}</el-col>
 </el-row>
<el-divider></el-divider>
<div>【题干】:<br/>
{{prevList.question}}
</div>
<el-divider></el-divider>
【参考答案】:
<el-button type="danger">视频答案预览</el-button>
<el-divider></el-divider>
【答案解析】:
<el-divider></el-divider>
【题目备注】:
<span slot="footer" class="dialog-footer">
<el-button type="primary" @click="handleClose">关 闭</el-button>
</span>
</el-dialog>
</div> -->

<script>

import { detail as questionsListById } from '@/api/hmmm/questions.js'
import { list } from '@/api/hmmm/directorys'
import { list as subList } from '@/api/hmmm/subjects'
export default {
  name: 'questionPreview',
  props: {
    id: {
      type: [Number, String, Object],
      required: true
    },
    dialogTableVisible: {
      type: Boolean,
      required: true
    }
  },
  data () {
    return {
      prevList: {},
      directory: '',
      subject: ''
    }
  },
  created () {
    this.getList()
  },
  methods: {
    async getList (id, oldId) {
      console.log(id, this.dialogTableVisible)
      // 获取详细试题
      const { data: quelist } = await questionsListById(id)
      this.prevList = quelist.data
      // 获取学科数据
      const { data: sub } = await subList(id)
      this.subject = sub.subjectName
      // 获取目录数据
      const { data: List } = await list()
      this.directory = List.directoryName
    },
    handelClose (done) {
      this.$emit('update:dialogTableVisible', false)
    }
  }
}
</script>

<style scoped lang='scss'></style>
