<template>
  <div class="JNPF-common-layout">
    <div class="JNPF-common-layout-left">
      <div class="JNPF-common-title">
        <h2>项目分类</h2>
      </div>
      <el-tree :data="industryTypeList" :props="defaultProps" default-expand-all highlight-current
        ref="treeBox" :expand-on-click-node="false" @node-click="handleNodeClick"
        class="JNPF-common-el-tree" node-key="id">
        <span class="custom-tree-node" slot-scope="{ node, data }">
          <i :class="data.icon"></i>
          <span class="text">{{node.label}}</span>
        </span>
      </el-tree>
    </div>
    <div class="JNPF-common-layout-center">
      <el-row class="JNPF-common-search-box" :gutter="16">
        <el-form @submit.native.prevent>
          <el-col :span="6">
            <el-form-item label="关键词">
              <el-input v-model="keyword" placeholder="请输入关键词查询" clearable />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item>
              <el-button type="primary" icon="el-icon-search" @click="search()">
                {{$t('common.search')}}</el-button>
              <el-button icon="el-icon-refresh-right" @click="refresh()">{{$t('common.reset')}}
              </el-button>
            </el-form-item>
          </el-col>
        </el-form>
      </el-row>
      <div class="JNPF-common-layout-main JNPF-flex-main">
        <div class="JNPF-common-head">
          <div></div>
          <div class="JNPF-common-head-right">
            <el-tooltip effect="dark" :content="$t('common.refresh')" placement="top">
              <el-link icon="icon-ym icon-ym-Refresh JNPF-common-head-icon" :underline="false"
                @click="refresh()" />
            </el-tooltip>
            <screenfull />
          </div>
        </div>
        <JNPF-table v-loading="listLoading" :data="list">
          <el-table-column prop="projectName" label="项目名称" sortable width="200" />
          <el-table-column prop="projectCode" label="项目编码" sortable width="160" />
          <el-table-column prop="projectPhase" label="项目阶段" sortable width="120" />
          <el-table-column prop="customerName" label="客户名称" sortable width="200" />
          <el-table-column prop="principal" label="负责人" sortable width="80" />
          <el-table-column prop="jackStands" label="立项人" sortable width="80" />
          <el-table-column label="交互时间" sortable width="100">
            <template slot-scope="scope">
              {{ scope.row.interactionDate | toDate("yyyy-MM-dd") }}
            </template>
          </el-table-column>
          <el-table-column prop="costAmount" label="费用金额" sortable width="100" />
          <el-table-column prop="tunesAmount" label="已用金额" sortable width="100" />
          <el-table-column prop="projectedIncome" label="预计收入" sortable width="100" />
          <el-table-column prop="registrant" label="登记人" sortable width="80"
            show-overflow-tooltip />
          <el-table-column label="登记时间" sortable width="120">
            <template slot-scope="scope">
              {{ scope.row.registerDate | toDate() }}
            </template>
          </el-table-column>
          <el-table-column prop="description" label="备注" sortable width="300" />
        </JNPF-table>
        <pagination :total="total" :page.sync="listQuery.currentPage" :limit.sync="listQuery.rows"
          @pagination="initData" />
      </div>
    </div>
  </div>
</template>

<script>
import { TableExampleListByType, TableExampleDelete } from '@/api/extend/tableExample'
export default {
  name: 'extend-tableDemo-treeTable',
  data() {
    return {
      keyword: '',
      list: [],
      total: 0,
      listLoading: true,
      listQuery: {
        currentPage: 1,
        pageSize: 20,
        sort: 'desc'
      },
      industryTypeList: [],
      defaultProps: {
        children: 'children1',
        label: 'fullName'
      },
      industryTypeId: "",
      industryTypeText: "",
    }
  },
  created() {
    this.getDictionaryData()
  },
  filters: {
    getTypeText(id, industryTypeList) {
      let item = industryTypeList.filter(o => o.id == id)[0]
      return item && item.fullName ? item.fullName : ''
    }
  },
  methods: {
    getDictionaryData() {
      this.$store.dispatch('base/getDictionaryData', { sort: 'IndustryType' }).then((res) => {
        this.industryTypeList = res
        this.$nextTick(() => {
          this.industryTypeId = this.industryTypeList[0].id
          this.industryTypeText = this.industryTypeList[0].fullName
          this.$refs.treeBox.setCurrentKey(this.industryTypeId);
          this.initData()
        });
      })
    },
    refresh() {
      this.keyword = ''
      this.search()
    },
    search() {
      this.listQuery = {
        currentPage: 1,
        pageSize: 20,
        sort: 'desc'
      }
      this.initData()
    },
    initData() {
      this.listLoading = true
      let query = {
        ...this.listQuery,
        keyword: this.keyword
      }
      TableExampleListByType(query, this.industryTypeId).then(res => {
        this.list = res.data.list
        this.total = res.data.pagination.total
        this.listLoading = false
      })
    },
    handleNodeClick(data) {
      if (this.industryTypeId == data.id) return
      this.industryTypeId = data.id
      this.industryTypeText = data.fullName
      this.initData()
    }
  }
}
</script>