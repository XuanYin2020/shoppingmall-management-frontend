<template>
  <div class="container">
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽">
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">
       批量保存
    </el-button>
    <el-button type="danger" @click="batchDelete">
       批量删除
    </el-button>
    <el-tree :data="menus" 
             show-checkbox node-key="catId" 
             :props="defaultProps"  
             :expand-on-click-node="false"
             :default-expanded-keys="expandedKeys"
             :draggable="draggable"
             :allow-drop="allowDrop"
             @node-drop="handleDrop"
             ref='menuTree'
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
      maxDepth: 1, // 为了拖拽效果中，判断深度
      updateNodes: [], // 拖拽效果中，需要修改的node的list
      currentParentCid: [], // 当前拖拽节点最新的父节点id
      draggable: false, // 是否开启拖拽的效果，和switch开关绑定
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
    },
    /** 拖拽效果：
     *  返回值：通过返回true或者false来判断能否被拖拽
     *  输入：draggingNode指的是被拖拽的menu，dropNode是目标menu，
     *        type指的是参数有三种情况：'prev'、'inner' 和 'next'，分别表示放置在目标节点前、插入至目标节点和放置在目标节点后
     *  核心：总层级数不能超过3，即被拖动的当前节点总层数 + 所在的父节点外层层级数<=3
     * */
    allowDrop (draggingNode, dropNode, type) {
      console.log('draggingNode', draggingNode, dropNode, type)
      // 统计被拖动的当前节点总层数
      this.maxDepth = draggingNode.data.catLevel
      this.countNodeLevel(draggingNode)
      // 当前正在拖动的节点 + 父节点所在是的深度不大于3即可
      let current = Math.abs(this.maxDepth - draggingNode.level + 1)  // 在批量保存的情况下，使用的是真正的信息，而不是根据数据库中的信息进行的修改
      console.log('this.maxDepth', this.maxDepth)
      console.log('深度：', current)
      if (type === 'inner') {
        return (current + dropNode.level) <= 3
      } else {
        return (current + dropNode.parent.level) <= 3
      }
    },
    /** 拖拽效果中的helper function */
    // 递归找到当前node下的最大深度
    countNodeLevel (node) {
      // 找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) { // tree的真正子节点
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxDepth) {
            this.maxDepth = node.childNodes[i].level
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    /**  拖拽数据收集 */
    handleDrop (draggingNode, dropNode, dropType, ev) {
      console.log('tree drop: ', draggingNode, dropNode, dropType)
      // 所需要的所有信息都可以从dropNode中获取到
      // 1. 当前拖拽节点最新的父节点id: 对于'prev'和 'next'父节点是dropNode的parent，'inner'父节点的id是dropNode
      // 2. 当前拖拽节点的顺序：->'inner' 兄弟节点是dropNode的child;
      // 3. 当前拖拽节点的level:当前的拖拽节点的层级发生变化的时候，所有子节点都需要发送改变
      // 将要修改的node信息，封装到this.updateNodes数据中
      let currentParentCid = 0
      let siblings = null
      if (dropType === 'inner') {
        currentParentCid = dropNode.data.catId
        siblings = dropNode.childNodes
      } else {
        // undefined是因为根节点的父节点是个数字，会返回个undefined
        currentParentCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      }
      // 2.遍历当前拖拽节点的父节点的所有子节点重新从0标记一下
      for (let i = 0; i < siblings.length; i++) {
        // 如果当前sibling的节点是拖拽的节点，还需要更改父节点信息parentCid (1)
        if (siblings[i].data.catId === draggingNode.data.catId) {
          let catLevel = draggingNode.level
          // 当前节点的层级发生变化  (2)
          if (siblings[i].level !== catLevel) {
            // 当前节点的层级发生变化
            catLevel = siblings[i].level
            // 修改子节点的层级
            this.updateChildNodeLevel(siblings[i])
          }
          // 改兄弟的顺序sort, 改变层级信息catLevel  (3)
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i, parentCid: currentParentCid, catLevel: catLevel})
        } else {
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i})// 改兄弟的顺序
        }
      }
      this.currentParentCid.push(currentParentCid)
    },
    /** 递归关系节点的level */
    updateChildNodeLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var childNode = node.childNodes[i].data // 真正的待更新的数据
          this.updateNodes.push({catId: childNode.catId, catLevel: childNode.catLevel = node.childNodes[i].level})
          // 递归处理子节点
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    },
    /** 批量的保存 */
    batchSave () {
      // 前端实现：发送http请求携带被修改的数据
      this.$http({
        url: this.$http.adornUrl('/product/category//update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({ // 顺序修改成功之后，弹出删除成功的弹框
          message: `菜单顺序修改成功`,
          type: 'success'
        })
        // 设置需要默认展开的node
        this.expandedKeys = this.currentParentCid
        // 前端需要刷新页面
        this.getMenus()
        // 清空拖拽的效果:之前保存数据的变量，应该变为默认值
        this.updateNodes = []
        this.maxDepth = 0
        this.currentParentCid = []
      })
      console.log('updateNodes:', this.updateNodes)
    },
    /** 批量删除 */
    batchDelete () {
      let catId = []
      let catName = []
      // 找到选中的所有节点信息
      let checkedNodes = this.$refs.menuTree.getCheckedNodes()
      console.log('被选中的元素', checkedNodes)
      // 遍历所有节点，获得id
      for (let i = 0; i < checkedNodes.length; i++) {
        catId.push(checkedNodes[i].catId)
        catName.push(checkedNodes[i].name)
      }
      // 3. 弹出删除的提示框
      this.$confirm(`是否批量删除: ${catName}?`, '提示', {
        confirmButtonText: 'ok',
        cancelButtonText: 'decline',
        type: 'warning'
      }).then(() => {
        // 点击了ok之后，发送http的删除请求
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(catId, false)
        }).then(({ data }) => {
          this.$message({ // 删除成功
            message: `菜单批量删除成功`,
            type: 'success'
          })
          this.getMenus()
        })
      }).catch(() => {})
    }
  },
  created () {
    this.getMenus()
  }
}
</script>

<style scoped lang="scss">
</style>
