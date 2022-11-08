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
        </span>
      </span>
    </el-tree>
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose">
      <span>这是一段信息</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
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
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    // 获得所有的三级目录内容
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
    // 发送请求实现
    append (data) {
      console.log('append', data)
    },
    // node是节点信息，data是数据信息
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
          // 5. 设置需要默认展开的node
          this.expandedKeys = [data.parent.data.catId]
          // 2. 前端需要刷新页面
          this.getMenus()
        })
      }).catch(() => {
      })
      console.log('remove', node, data)
    }

  },
  created () {
    this.getMenus()
  }
}
</script>

<style scoped lang="scss">
</style>
