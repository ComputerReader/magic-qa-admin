<template>
  <el-card>
      <!-- 头部 搜索栏 -->
    <div>
        <el-row type="flex">
          <el-col :span="4">
            <el-input
              placeholder="关键词"
              v-model="queryForm.keyword"
              @keypress.native.enter="handleSearchBtnClick"
              size="small"
            ></el-input>
          </el-col>
          <el-col :span="2">
            <el-button
              plain
              type="primary"
              icon="el-icon-search"
              size="small"
              @click="handleSearchBtnClick"
            >搜索</el-button>
          </el-col>
          <el-col :span="2">
            <el-button
              plain
              type="primary"
              icon="el-icon-refresh-right"
              size="small"
              @click="handleResetBtnClick"
            >重置</el-button>
          </el-col>
        </el-row>
    </div>
    <!-- 主体 查询表 -->
    <el-table
      class="listTable"
      :data="tableData"
      :header-cell-style="{
          background: '#e4f1e2',
          color: '#606266',
          'text-align': 'center'
        }"
      :cell-style="{ 'text-align': 'center', border: 'none' }"
      stripe
    >
      <el-table-column prop="from_name" label="回复人" width="250">
        <template slot-scope="scope">
          <span>{{ scope.row.from_name }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="to_name" label="被回复人" width="250">
        <template slot-scope="scope">
          <span>{{ scope.row.to_name }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="context" label="内容" width="500">
        <template slot-scope="scope">
          <span>{{ scope.row.context }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="from_state" label="回复人发言身份" width="120">
        <template slot-scope="scope">
          <span>{{ scope.row.from_state }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="to_state" label="被回复人发言身份" width="120">
        <template slot-scope="scope">
          <span>{{ scope.row.to_state }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="time" label="时间" width="200">
        <template slot-scope="scope">
          <span>{{ timestampToTime(scope.row.time) }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="post_id" label="回复详情">
        <template slot-scope="scope">
            <el-button icon="el-icon-edit-outline" type="text" @click="onClickDetailButton(scope.row.post_id)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 尾部 分页栏 -->
      <div align="center">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryForm.currentPage"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="queryForm.size"
          layout="total, sizes, prev, pager, next, jumper"
          :total="queryForm.total"
        ></el-pagination>
      </div>
  </el-card>
</template>

<script>
import time from '../../../mixins/time'
import { getReplyInfo } from '../../../api/requsetMethods'
export default {
  name: 'qa-replyInfo',
  mixins: [time],
  created () {
    console.log('跳转主页后', localStorage.getItem('accessToken'), localStorage.getItem('refreshToken'))
    this.updateTable()
  },
  mounted () {
    window.Vue = this
  },
  methods: {
    // 更新列表
    updateTable () {
      getReplyInfo(this.queryForm)
        .then(res => {
          console.log(res.data)
          const statusCode = res.data.status_code
          switch (statusCode) {
            case 361:
              console.log('成功', res.data)
              this.tableData = res.data.replyInfo
              this.queryForm.total = res.data.total
              break
            case 362:
              alert('获取用户数据失败')
              break
          }
        })
        .catch(error => {
          console.log('失败', error)
        })
    },
    // 跳转到 帖子详情 页面
    onClickDetailButton (id) {
      // this.$router.push({ path: this.$route.path + '/detail', query: { uid: id } })
    },
    // 处理搜索按钮点击事件
    async handleSearchBtnClick () {
      this.updateTable()
    },
    // 处理重置按钮点击事件
    async handleResetBtnClick () {
      this.queryForm.keyword = ''
      this.queryForm.currentPage = 1
      this.queryForm.pageSize = 10
      this.updateTable()
    },
    // 处理页面值变化事件
    async handleSizeChange (val) {
      this.queryForm.pageSize = val
      this.updateTable()
    },
    // 处理当前页跳转事件
    async handleCurrentChange (val) {
      this.queryForm.currentPage = val
      this.updateTable()
    }
  },
  data () {
    return {
      // 查询条件、分页信息
      queryForm: {
        keyword: '',
        total: 1,
        currentPage: 1,
        pageSize: 10
      },
      // 信息: name, title，context, state，time
      tableData: []
    }
  }
}
</script>

<style>
.listTable {
  margin-top:1%;
  margin-bottom:1%;
  font-size: 12px;
  /* color: #56a286; */
  width: 100%;
}
</style>
