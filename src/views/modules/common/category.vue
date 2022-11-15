<template>
  <el-tree :data="menus" :props="defaultProps" node-key="catId" ref="menuTree" @node-click="nodeclick">
  </el-tree>
</template>

<script>
export default {
  data() {
    return {
      menus: [], //绑定所有菜单数据
      expandedKeys: [], // 默认展开为空
      dialogVisible: false,
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods:{
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'), // 请求的地址
        method: 'get'
      }).then(({data}) => { // 请求成功
        console.log('成功获取到菜单数据', data.data)
        this.menus = data.data
      })
    },
    nodeclick (data,node,component) {
        console.log("子组件的节点被点击",data,node,component)
        //向父组件发送事件(第一个参数：事件的名字；后面任意长度的参数)
        this.$emit("tree-node-click",data,node,component)
    }
  },
  components: {},
  // 生命周期-创建完成
  created () {
    this.getMenus()//绑定从后端获取到的 菜单数据
  }
};
</script>

<style scoped lang="scss">
</style>
