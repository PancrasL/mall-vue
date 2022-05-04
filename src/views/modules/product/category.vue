<template>
  <div>
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>

    <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox
             node-key="catId" :default-expanded-keys="expandedKey">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level<=2" type="text" size="mini"
                     @click="() => append(data)">Append</el-button>
          <el-button v-if="node.childNodes.length===0" type="text" size="mini"
                     @click="() => remove(node, data)">Delete</el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>

<script>
export default {
  data () {
    return {
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0},
      menus: [],
      expandedKey: [],
      dialogVisible: false,
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    // 获取商品分类列表
    getMenus () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    // 添加树形条目
    append (data) {
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel + 1
      console.log(this.category)
    },
    // 删除树形条目
    remove (node, data) {
      this.$confirm(`此操作将删除[${data.name}]分类, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let ids = [data.catId]
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(() => {
          this.$message({
            message: '删除成功',
            type: 'success'
          })
          this.getMenus()
          // 设置需要默认展开的菜单
          this.expandedKey = [node.parent.data.catId]
        })
      }).catch(() => {
        this.$message({
          message: '取消成功'
        })
      })
    },
    // 添加分类
    addCategory () {
      console.log(this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: '分类菜单添加成功',
          type: 'success'
        })
        this.dialogVisible = false
        this.getMenus()
        // 设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid]
      })
    }
  },
  // 生命周期-创建完成
  created () {
    this.getMenus()
  },
  mounted () {
  },
  beforeCreate () {
  },
  beforeMount () {
  },
  beforeUpdate () {
  },
  updated () {
  },
  beforeDestroy () {
  },
  destroyed () {
  },
  activated () {
  }
}
</script>

<style scoped>

</style>


