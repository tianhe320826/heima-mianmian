<template>
  <div class="companys-container paddingContainer">
    <el-card shadow="never">
      <!-- 学科名称搜索 -->
      <el-row :gutter="10" type="flex" justify="space-around">
        <el-form ref="companysQueryInfoRef" label-position="top" :model="companysQueryInfo">
          <el-col :span="5">
            <el-form-item label="标签名称">
              <el-input
                placeholder="请输入"
                class="filter-item"
                v-model="companysQueryInfo.tags"
                clearable
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="3">
            <el-form-item label="省">
              <el-select
                v-model="companysQueryInfo.province"
                placeholder="请选择"
                clearable
                @change="getAreasF($event)"
              >
                <el-option
                  v-for="(item,index) in provinces"
                  :key="index"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="3">
            <el-form-item label="市">
              <el-select v-model="companysQueryInfo.city" placeholder="请选择" clearable>
                <el-option v-for="(item,index) in Areas" :key="index" :label="item" :value="item"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="5">
            <el-form-item label="企业简称">
              <el-input class="filter-item" clearable v-model="companysQueryInfo.shortName"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="3">
            <el-form-item label="状态">
              <el-select placeholder="请选择" v-model="companysQueryInfo.state" clearable>
                <el-option label="启用" value="1"></el-option>
                <el-option label="禁用" value="0"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="2">
            <el-form-item label="操作">
              <el-button type="primary" icon="el-icon-search" @click="searchList">搜索</el-button>
            </el-form-item>
          </el-col>
        </el-form>
      </el-row>
      <el-button
        type="primary"
        icon="el-icon-edit"
        class="addUser-btn"
        @click="addCompanysFormShow"
      >新增用户</el-button>
      <el-alert :title="alertText" type="info" class="alert" :closable="false" show-icon></el-alert>
      <!-- end -->
      <!--企业数据-->
      <el-table
        :data="companysDataList"
        row-key="id"
        fit
        highlight-current-row
        style="width: 100%"
        border
      >
        <el-table-column :label="$t('table.id')" type="index"></el-table-column>
        <!-- <el-table-column align="center" label="序号" prop="id"></el-table-column> -->
        <el-table-column label="企业编号">
          <template slot-scope="scope">
            <span>{{scope.row.number}}</span>
          </template>
        </el-table-column>
        <el-table-column label="企业简称">
          <template slot-scope="scope">
            <span>{{scope.row.shortName}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.tag')">
          <template slot-scope="scope">
            <span>{{scope.row.tags}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.creatdate')">
          <template slot-scope="scope">
            <span>{{scope.row.addDate|parseTimeByString("{y}-{m}-{d} {h}:{i}:{s}")}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.remarks')">
          <template slot-scope="scope">
            <span>{{scope.row.remarks}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.state')">
          <template slot-scope="scope">
            <span v-if="scope.row.state===1">启用</span>
            <span v-if="scope.row.state===0">禁用</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('table.actions')" min-width="150" class="action-btn">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              @click="showEditCompanysForm(scope.row)"
            >{{$t('table.edit')}}</el-button>
            <!-- <el-button
              type="primary"
              size="mini"
              @click="changeState(scope.row)"
            >{{scope.row.state?$t('table.prohibit'):"启用"}}</el-button>-->
            <el-button
              type="primary"
              size="mini"
              native-type="button"
              @click="changeState(scope.row,$event)"
            >{{scope.row.state?$t('table.prohibit'):'启用'}}</el-button>
            <el-button
              type="danger"
              size="mini"
              @click="removeItems(scope.row)"
            >{{$t('table.delete')}}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="companysQueryInfo.page"
          :page-sizes="[5,10, 15, 20]"
          :page-size="companysQueryInfo.pagesize"
          layout="sizes,prev, pager, next, jumper"
          :total="total"
        ></el-pagination>
      </div>
    </el-card>
    <!-- 新增用户区域 -->
    <el-dialog
      title="创建用户"
      :visible.sync="addCompanysFormVisible"
      width="50%"
      @close="clearAddCompanyForm"
    >
      <el-form
        :model="addCompanysForm"
        :rules="addCompanysFormRule"
        ref="addCompanysFormRef"
        label-width="100px"
      >
        <el-form-item label="企业名称" required prop="shortName">
          <el-input v-model="addCompanysForm.shortName"></el-input>
        </el-form-item>
        <el-form-item>
          <el-checkbox label="是否为名企" v-model="addCompanysForm.isFamous"></el-checkbox>
        </el-form-item>
        <el-form-item label="所属公司" prop="company">
          <el-input v-model="addCompanysForm.company"></el-input>
        </el-form-item>
        <el-form-item>
          <el-link
            href="https://element.eleme.io"
            target="_blank"
          >https://www.tianyancha.com (在此可查所属公司全称及简称)</el-link>
        </el-form-item>
        <el-form-item label="城市" required>
          <el-form-item prop="province" class="city-items">
            <!-- 省区选择 -->
            <el-select
              v-model="addCompanysForm.province"
              placeholder="请选择"
              clearable
              @change="getAreasT($event)"
            >
              <el-option v-for="(item,index) in provinces" :key="index" :label="item" :value="item"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item prop="city" class="city-items">
            <!-- 城市选择 -->
            <el-select v-model="addCompanysForm.city" placeholder="请选择" clearable>
              <el-option v-for="(item,index) in Areas" :key="index" :label="item" :value="item"></el-option>
            </el-select>
          </el-form-item>
        </el-form-item>
        <el-form-item label="方向(企业标签)" required prop="tags">
          <el-input v-model="addCompanysForm.tags"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="remarks">
          <el-input type="textarea" placeholder="请输入" v-model="addCompanysForm.remarks"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addCompanysFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCompanys">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 是否启用或者禁用对话框 -->
    <!-- 编辑对话框区域 -->
    <el-dialog
      title="编辑用户"
      :visible.sync="editCompanysFormVisible"
      width="50%"
      @close="clearEditCompanyForm"
    >
      <el-form
        :model="editCompanysForm"
        :rules="editCompanysFormRule"
        ref="editCompanysFormRef"
        label-width="100px"
      >
        <el-form-item label="企业名称" required prop="shortName">
          <el-input v-model="editCompanysForm.shortName"></el-input>
        </el-form-item>
        <el-form-item>
          <el-checkbox label="是否为名企" v-model="editCompanysForm.isFamous"></el-checkbox>
        </el-form-item>
        <el-form-item label="所属公司" prop="company">
          <el-input v-model="editCompanysForm.company"></el-input>
        </el-form-item>
        <el-form-item>
          <el-link
            href="https://element.eleme.io"
            target="_blank"
          >https://www.tianyancha.com (在此可查所属公司全称及简称)</el-link>
        </el-form-item>
        <el-form-item label="城市" required>
          <el-form-item prop="province" class="city-items">
            <!-- 省区选择 -->
            <el-select
              v-model="editCompanysForm.province"
              placeholder="请选择"
              clearable
              @change="getAreasT($event)"
            >
              <el-option v-for="(item,index) in provinces" :key="index" :label="item" :value="item"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item prop="city" class="city-items">
            <!-- 城市选择 -->
            <el-select v-model="editCompanysForm.city" placeholder="请选择" clearable>
              <el-option v-for="(item,index) in Areas" :key="index" :label="item" :value="item"></el-option>
            </el-select>
          </el-form-item>
        </el-form-item>
        <el-form-item label="方向(企业标签)" required prop="tags">
          <el-input v-model="editCompanysForm.tags"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="remarks">
          <el-input type="textarea" placeholder="请输入" v-model="editCompanysForm.remarks"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editCompanysFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCompanys">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { list, disabled, remove, detail, update, add } from '@/api/hmmm/companys.js'
import { provinces, citys } from '@/api/hmmm/citys.js'
export default {
  components: {},
  props: {},
  data () {
    return {
      companysDataList: [],
      companysQueryInfo: {
        page: 1,
        pagesize: 5,
        tags: '',
        province: '',
        city: '',
        shortName: '',
        state: ''
      },
      total: 10,
      alertText: '',
      addCompanysFormVisible: false,
      addCompanysForm: {
        isFamous: '',
        shortName: '',
        company: '',
        province: '',
        city: '',
        tags: '',
        remarks: ''
      },
      addCompanysFormRule: {
        shortName: [
          { required: true, message: '请输入公司名称', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 3到15个字符', trigger: 'blur' }
        ],
        company: [{ min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }],
        tags: [{ required: true, message: '请输入企业标签', trigger: 'blur' }],
        city: [{ required: true, message: '请输入城市名', trigger: 'change' }],
        province: [{ required: true, message: '请输入省或者直辖市名', trigger: 'change' }]
      },
      Areas: [],
      currentProvince: '',
      currentArea: '',
      editCompanysFormVisible: false,
      editCompanysForm: {},
      editCompanysFormRule: {
        shortName: [
          { required: true, message: '请输入公司名称', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 3到15个字符', trigger: 'blur' }
        ],
        company: [{ min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }],
        tags: [{ required: true, message: '请输入企业标签', trigger: 'blur' }],
        city: [{ required: true, message: '请输入城市名', trigger: 'change' }],
        province: [{ required: true, message: '请输入省或者直辖市名', trigger: 'change' }]
      }
    }
  },
  computed: {
    provinces: function () {
      return provinces()
    }
  },
  created () {
    this.getCompanysList()
  },
  methods: {
    //   获取公司数据列表
    async getCompanysList () {
      try {
        const { data } = await list(this.companysQueryInfo)
        this.companysDataList = data.items
        this.total = data.counts
        this.alertText = '共' + data.counts + '条记录'
      } catch (err) {
        this.$message.error('获取公司列表失败！')
      }
    },
    searchList () {
      this.getCompanysList()
    },
    handleSizeChange (val) {
      this.companysQueryInfo.pagesize = val
      this.getCompanysList()
    },
    handleCurrentChange (val) {
      this.companysQueryInfo.page = val
      this.getCompanysList()
    },
    // 显示新增公司的弹出框
    addCompanysFormShow () {
      this.currentArea = ''
      this.currentProvince = ''
      this.addCompanysFormVisible = true
    },
    // 新增公司用户
    addCompanys () {
      this.$refs.addCompanysFormRef.validate(async valid => {
        if (valid) {
          try {
            this.addCompanysForm.isFamous = !!this.addCompanysForm.isFamous
            const { data } = await add(this.addCompanysForm)
            console.log(data)
            this.$message.success('添加成功')
            this.getCompanysList()
            this.addCompanysFormVisible = false
          } catch (err) {
            this.$message.error('添加失败！')
            this.addCompanysFormVisible = false
          }
        }
      })
    },
    // 关闭新增对话框，清空表单
    clearAddCompanyForm () {
      this.$refs.addCompanysFormRef.resetFields()
    },

    // 启用功能，状态改变
    changeState (row, e) {
      this.$confirm('已成功启用, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.companysQueryInfo.tags = ''
        this.companysQueryInfo.province = ''
        this.companysQueryInfo.city = ''
        this.companysQueryInfo.shortName = ''
        this.companysQueryInfo.state = null
        const params = {
          id: row.id,
          state: Math.abs(row.state - 1)
        }
        disabled(params).then(data => {
          if (data.status === 200) {
            this.$message({
              type: 'success',
              message: '已启用成功!'
            })
            this.getCompanysList()
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消操作'
        })
      })
    },
    // 根据省获取市和地区数据
    getAreasF (pro) {
      this.companysQueryInfo.city = ''
      this.Areas = citys(pro)
    },
    getAreasT (pro) {
      this.addCompanysForm.city = ''
      this.Areas = citys(pro)
    },
    // 删除指定用户功能
    removeItems (row) {
      this.$confirm('此操作将永久删除用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        remove(row).then(data => {
          this.getCompanysList()
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    // 退出编辑时清空
    clearEditCompanyForm () {
      this.$refs.editCompanysFormRef.validate(vali => {
        if (vali) {
          this.$refs.editCompanysFormRef.resetFields()
        }
      })
    },
    // 显示用户编辑对话框
    async showEditCompanysForm (row) {
      try {
        const { data } = await detail(row)
        console.log(data)
        this.editCompanysForm = data
        // this.editCompanysForm.isFamous = !!this.editCompanysForm.isFamous
        // this.editCompanysForm.id = this.editCompanysForm.id + ''
        // this.editCompanysForm.number = null
        this.editCompanysFormVisible = true
      } catch (err) {
        this.$message.error('操作失败！')
      }
    },
    // 编辑用户信息区域
    async editCompanys () {
      try {
        const params = {
          id: this.editCompanysForm.id,
          number: this.editCompanysForm.number,
          isFamous: !!this.editCompanysForm.isFamous,
          shortName: this.editCompanysForm.shortName,
          company: this.editCompanysForm.company,
          province: this.editCompanysForm.province,
          city: this.editCompanysForm.city,
          tags: this.editCompanysForm.tags,
          remarks: this.editCompanysForm.remarks
        }
        console.log(params)
        console.log(this.editCompanysForm)
        const { data } = await update(params)
        this.getCompanysList()
        this.editCompanysFormVisible = false
      } catch (err) {
        this.$message.error('编辑用户信息失败！')
        this.editCompanysFormVisible = false
      }
    }

  },
  mounted () { }
}
</script>
<style lang="scss" scoped>
.pagination {
  margin-top: 10px;
}
.addUser-btn {
  margin-bottom: 10px;
  float: right;
}
.dialog-footer {
  text-align: center;
}
.city-items {
  float: left;
}
</style>
