<template>
  <el-tree 
  show-checkbox 
  node-key="catId" 
  :data="menus" 
  :expand-on-click-node="true" 
  :highlight-current="true">
    <span class="custom-tree-node" slot-scope="{node,data}">
      <span>{{ node.data.name }}</span>
      <span>
        <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
          添加
        </el-button>
        <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
          删除
        </el-button>
      </span>
    </span>
  </el-tree>
</template>


<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json 文件，图片文件等等）
//例如：import 《组件名称》from '《组件路径》';

export default {
  //import 引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    return {
      menus: [],
      defaultProps: {
        children: 'children',//子节点属性名
        label: 'name'//节点展示名
      }
    };
  },
  //计算属性类似于data 概念
  computed: {},
  //监控data 中的数据变化
  watch: {},
  //方法集合
  methods: {

    // 打开页面时加载数据
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(({ data }) => {
        console.log("成功获取到菜单数据！", data.data)
        this.menus = data.data; //将获取的值赋值给data中的menus
      })
    },
    // 添加节点
    append(data) {
      console.log("append...");
    },
    // 移除节点
    remove(node, data) {
      console.log("remove", node, data);
    }

  },
  //生命周期- 创建完成（可以访问当前this 实例）
  created() {
    // 打开页面时加载数据
    this.getMenus();
  },
  //生命周期- 挂载完成（可以访问DOM 元素）
  mounted() {

  },
  beforeCreate() { }, //生命周期- 创建之前
  beforeMount() { }, //生命周期- 挂载之前
  beforeUpdate() { }, //生命周期- 更新之前
  updated() { }, //生命周期- 更新之后
  beforeDestroy() { }, //生命周期- 销毁之前
  destroyed() { }, //生命周期- 销毁完成
  activated() { }, //如果页面有keep-alive 缓存功能，这个函数会触发 
}
</script>
<style lang='scss' scoped>
//@import url(); 引入公共css 类
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>
