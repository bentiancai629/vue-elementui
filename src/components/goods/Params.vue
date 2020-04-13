<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数" type="warning" :closable="false"></el-alert>
    </el-card>

    <!-- 选择商品分类区域 -->
    <el-row class="cat_opt">
      <el-col>
        <span>商品分类</span>
        <!-- 选择商品分类级联选择器 -->
        <el-cascader
          v-model="selectedCateKeys"
          :options="catelist"
          :props="{ expandTrigger: 'hover' }"
          @change="handleChange"
        ></el-cascader>
      </el-col>
    </el-row>

    <!-- tab 页签区域 -->
    <el-tabs v-model="activeName" @tab-click="handleTabClick">
      <!-- 添加动态参数的面板 -->
      <el-tab-pane label="动态参数" name="many">
        <!-- 添加参数按钮 -->
        <el-button type="primary" size="mini" :disabled="isBtnDisabled">添加参数</el-button>
        <!-- 动态参数表格 -->
        <el-table :data="manyTableData" border stripe>
           <el-table-column type="expand"></el-table-column>
          <!-- 索引列 -->
          <el-table-column type="index"></el-table-column>
          <el-table-column label="参数名称" prop="attr_name"></el-table-column>
          <el-table-column label="操作" >
            <template >
              <el-button size="mini" type="primary" icon="el-icon-edit">搜索</el-button>
              <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <!-- 添加静态属性的面板 -->
      <el-tab-pane label="静态属性" name="only">
        <!-- 添加属性按钮 -->
        <el-button type="primary" size="mini" :disabled="isBtnDisabled">添加属性</el-button>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类列表
      catelist: [],
      cateProps: { value: 'cat_id', label: 'cat_name', children: 'children' },
      // 级联选择框双向绑定到的数组
      selectedCateKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      manyTableData: [],
      onlyTableData: []
    }
  },
  created() {
    this.getCateList()
  },
  // 计算属性
  computed: {
    // 如果按钮需要被禁用 返回true 返回false
    isBtnDisabled() {
      if (this.selectedCateKeys.lengh !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类id
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    }
  },
  methods: {
    // 获取所有的商品分类
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.catelist = res.data

      // console.log(this.catelist)
    },
    // 级联选择框选中项变化 触发函数
    async handleChange() {
      this.getParamsData()
    },
    // tab 页签点击事件的处理函数
    handleTabClick() {
      console.log(this.activeName)
      this.getParamsData()
    },
    // 获取参数列表数据
    async getParamsData() {
      // console.log(this.selectedCateKeys)
      // 证明选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        return
      }
      // 证明选中的是3级分类
      console.log(this.selectedCateKeys)
      // 根据所选的分类id和面板获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }

      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData.data = res.data
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px;
}
</style>
