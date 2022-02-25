<template>
  <el-tree
    :data="menus"
    node-key="catId"
    :props="defaultProps"
    ref="menuTree"
    @node-click="nodeclick"
  >
  </el-tree>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    //这里存放数据
    return {
      menus: [], // 菜单
      expandedKey: [], // 刷新展开菜单id
      defaultProps: {
        children: "children",
        label: "name", // 显示的标签
      },
    };
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    //   获取菜单 -- 发送请求模板
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        // 解构data
        // console.log(data.data)
        this.menus = data.data;
      });
    },
    // 点击回调
    nodeclick(data, node, componet) {
    //   console.log("数据，当前节点，当前组件：", data, node, componet);
    // 向父组件传递事件 事件名(任意) 任意数据
    this.$emit("tree-node-click", data, node, componet)
    },
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
/* @import url(); 引入公共css类 */
</style>