<template>
    <el-tree :data="menus" :props="defaultProps" @node-click="nodeClick" node-key="catId" ref="menuTree">
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
        //这里存放数据
        return {
            menus: [],
            expandedKeys: [],
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
        // ==================打开页面时加载数据==========================
        getMenus() {
            // 发送菜单树状结构请求
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                //将获取的值赋值给data中的menus
                this.menus = data.data;
            })
        },
        // ==================点击事件==================
        // node-click	节点被点击时的回调	
        // 共三个参数，依次为：传递给 data(属性的数组中该节点所对应的对象)、节点对应的:Node、节点组件本身:component。
        nodeClick(data, node, component) {
            // console.log("子节点被点击DB信息:", data, "层级信息:", node, "组件本身:", component);
            // 向父组件发送tree-node-click事件 传递三个参数
            this.$emit("tree-node-click", data, node, component);
            // 父组件感知tree-node-click事件
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
</style>