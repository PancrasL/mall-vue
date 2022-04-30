<template>
  <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
  data () {
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    // 获取商品分类列表
    getMenus () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        console.log('Sucess: ' + data)
      })
    }
  },
  // 生命周期-创建完成
  created () {
    this.getMenus()
  }
}
</script>

<style scoped>

</style>


