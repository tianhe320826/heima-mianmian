<template>
  <div class="subject-card">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">学科管理</el-breadcrumb-item>
      <el-breadcrumb-item>{{subjectName}}</el-breadcrumb-item>
      <el-breadcrumb-item>{{linkName}}</el-breadcrumb-item>
    </el-breadcrumb>
  </div>
</template>

<script>
export default {
  components: {},
  props: ['subjectName', 'linkName'],
  data () {
    return {
    }
  },
  watch: {},
  computed: {},
  methods: {},
  created () { },
  mounted () { }
}
</script>
<style lang="less" scoped>
</style>
