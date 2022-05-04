<template>
  <div>
    <el-dialog
      :title="dialog.title"
      :visible.sync="dialog.visible"
      width="30%"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialog.visible = false">取 消</el-button>
        <el-button type="primary" @click="submit">确 定</el-button>
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
          <el-button v-if="true" type="text" size="mini"
                     @click="() => edit(data)">Edit</el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>

<script>
export default {
  data () {
    return {
      category: {catId: null, name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, icon: '', productUnit: ''},
      menus: [],
      expandedKey: [],
      dialog: {title: null, visible: false, type: null},
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    // 获取商品分类列表
    getMenus (expandedKey) {
      // 默认展开的菜单
      this.expandedKey = expandedKey
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    // 添加分类信息
    append (data) {
      this.dialog = {title: '添加分类信息', visible: true, type: 'append'}
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel + 1
    },
    // 删除分类信息
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
          this.getMenus([node.parent.data.catId])
        })
      }).catch(() => {
        this.$message({
          message: '取消成功'
        })
      })
    },
    // 修改分类信息
    edit (data) {
      this.dialog = {title: '修改分类信息', visible: true, type: 'edit'}
      // 发送请求，获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(() => {
        this.category = {
          catId: data.catId,
          name: data.name,
          parentCid: data.parentCid,
          catLevel: data.catLevel,
          showStatus: data.showStatus,
          sort: data.sort,
          icon: data.icon,
          productUnit: data.productUnit
        }
        console.log('回显数据：', this.category)
      })
    },

    // 提交对话框信息
    submit () {
      if (this.dialog.type === 'append') {
        this.addCategory()
      } else if (this.dialog.type === 'edit') {
        this.editCategory()
      }
    },
    // 添加分类
    addCategory () {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(() => {
        this.$message({
          message: '分类菜单添加成功',
          type: 'success'
        })
        this.dialog.visible = false
        this.getMenus([this.category.parentCid])
      })
    },
    // 修改分类
    editCategory () {
      let {catId, name, icon, productUnit} = this.category
      let sendData = {catId, name, icon, productUnit}
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(sendData, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单修改成功',
          type: 'success'
        })
        this.dialog.visible = false
        this.getMenus([this.category.parentCid])
      })
    }
  },
  // 生命周期-创建完成
  created () {
    this.getMenus([])
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


