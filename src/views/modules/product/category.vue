<template>
  <div class="container">
    <el-tree :data="menus" 
             show-checkbox node-key="catId" 
             :props="defaultProps"  
             :expand-on-click-node="false"
             :default-expanded-keys="expandedKeys"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level<=2" type="text" size="mini" @click="() => append(data)"> Append </el-button>
          <el-button v-if="node.childNodes.length==0" type="text" size="mini" @click="() => remove(node, data)"> Delete </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">Edit </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal = 'false'
      >
      <el-form :model="category">
        <el-form-item label="Category Name">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <el-form :model="category">
        <el-form-item label="Category Icon">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <el-form :model="category">
        <el-form-item label="Category unit">
          <el-input v-model="category.productunit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      menus: [],
      expandedKeys: [], // 默认展开为空
      dialogVisible: false,
      // form表单和该数据绑定，意味着表单的数据对象，也就是整个表单中的所有对象和谁绑定
      category: { // 包含若干新项目的数据
        name: '',
        catId: null,
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productunit: null,
        icon: ''
      },
      dialogType: '', // 为了复用对话框，edit或者append
      dialogTitle: '',
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    /* 获得所有的三级目录内容 */
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'), // 请求的地址
        method: 'get'
        //   params: this.$http.adornParams({
        //     'page': this.pageIndex,
        //     'limit': this.pageSize,
        //     'roleName': this.dataForm.roleName
        //   })
      }).then(({data}) => { // 请求成功
        console.log('成功获取到菜单数据', data.data)
        this.menus = data.data
      })
    },
    /* 【新增】：发送请求实现n */
    append (data) {
      console.log('append', data)
      this.dialogVisible = true
      // 对话框title
      this.dialogTitle = 'Append new Category'
      // 复用对话框
      this.dialogType = 'append'
      // 在当前的菜单层级下，添加新的的item的信息，目的是保持原来tree打开的层级
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      // 在对话框中不显示回显任何内容
      this.category.name = ''
      this.category.catId = null
      this.category.icon = ''
      this.category.productunit = ''
    },
    // 添加三级分类
    addCategory () {
      console.log('提交的数据', this.category)
      // 添加http请求，保存当前新增的菜单
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({ // 添加成功之后，弹出删除成功的弹框
          message: `${this.category.name} Menu append successfully`,
          type: 'success'
        })
        // 关闭对话框
        this.dialogVisible = false
        // 设置需要默认展开的node
        this.expandedKeys = [this.category.parentCid]
        // 前端需要刷新页面
        this.getMenus()
      })
    },
    /* 【删除】功能：node是节点信息，data是数据信息 */
    remove (node, data) {
      // 3. 点击删除，先发送删除提示,弹出确认弹框
      this.$confirm(`It will delete the ${data.name} item, click the OK button to execute.`, '提示', {
        confirmButtonText: 'ok',
        cancelButtonText: 'decline',
        type: 'warning'
      }).then(() => {
        // 1. 发送http请求，POST类型的删除请求，请求体必须是json类型
        var ids = [data.catId]
        var name = data.name
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({ // 4.删除成功之后，弹出删除成功的弹框
            message: `${name} item delete successfully`,
            type: 'success'
          })
          // 2. 前端需要刷新页面
          this.getMenus()
          // 5. 设置需要默认展开的node
          this.expandedKeys = [node.parent.data.catId]
        })
      }).catch(() => {
      })
      console.log('remove', node, data)
    },
    /* 【修改】功能 */
    edit (data) {
      console.log('要修改的数据', data)
      // 对话框title
      this.dialogTitle = 'Edit Category'
      // 复用对话框
      this.dialogType = 'edit'
      this.dialogVisible = true
      // 回显信息：发送请求，获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        // 请求成功，回显数据
        console.log('要回回显的数据', data)
        this.category.name = data.data.name // 此data是从后端返回的数据
        this.category.catId = data.data.catId
        this.category.icon = data.data.icon
        this.category.productunit = data.data.productunit
        this.category.parentCid = data.data.parentCid
      })
    },
    // 真正提交修改三级分类
    editCategory () {
      // 将要发送的数据：不想更新的数据就不发
      var {catId, name, icon, productunit} = this.category
      var data = {catId: catId, name: name, icon: icon, productunit: productunit} // {catId, name, icon, productunit}
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(data, false)
      }).then(({ data }) => {
        this.$message({ // 修改成功之后，弹出删除成功的弹框
          message: `${this.category.name} Menu edit successfully`,
          type: 'success'
        })
        // 关闭对话框
        this.dialogVisible = false
        // 设置需要默认展开的node
        this.expandedKeys = [this.category.parentCid]
        // 前端需要刷新页面
        this.getMenus()
      })
    },
    /* 判断是【修改】还是【新增】，复用对话框 */
    submitData (data) {
      if (this.dialogType === 'append') {
        this.addCategory()
      }
      if (this.dialogType === 'edit') {
        this.editCategory()
      }
    }
  },
  created () {
    this.getMenus()
  }
}
</script>

<style scoped lang="scss">
</style>
