<template>
  <div class="container">
    <el-tree :data="menus" show-checkbox node-key="catId" :props="defaultProps"  :expand-on-click-node="false">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level<=2" type="text" size="mini" @click="() => append(data)"> Append </el-button>
          <el-button v-if="node.childNodes.length==0" type="text" size="mini" @click="() => remove(node, data)"> Delete </el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>

<script>
export default {
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
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
    remove (node, data) {
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
