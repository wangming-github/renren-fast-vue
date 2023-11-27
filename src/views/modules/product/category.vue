<template>
  <div>
    <el-tree show-checkbox node-key="catId" :data="menus" :default-expanded-keys="expandedKeys"
      :expand-on-click-node="true" :highlight-current="true" width="50%">
      <span class="custom-tree-node" slot-scope="{node,data}">
        <span>{{ node.data.name }}</span>
        <span>
          <!-- 当为父节点展示此按钮 -->
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            添加
          </el-button>
          <!-- 当为子节点展示此按钮 -->
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
            删除
          </el-button>
          <!-- 任何时候都展示此按钮 -->
          <el-button type="text" size="mini" @click="() => edit(data)">
            修改
          </el-button>
        </span>
      </span>
    </el-tree>

    <!-- dialog -->
    <el-dialog title="修改菜单信息" :visible.sync="centerDialogVisible" width="580px" center>
      <el-form :model="category" label-position="right" label-width="150px">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="centerDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>

  </div>
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
      category: {
        name: null,//
        catId: null,//
        productUnit: null,//
        icon: null,//
        parentCid: 0, //
        catLevel: 0, //
        showStatus: 1,//默认显示
        sort: 0
      },
      labelPosition: 'left',//Dialog对话框中表单对齐方式
      centerDialogVisible: false,//Dialog对话框默认关闭
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
    // ==========================添加节点==========================
    append(data) {
      console.log(" append...", data); this.$prompt('请输入菜单名称', '添加菜单', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
      }).then(({ value }) => {
        this.category.name = value;//菜单名称
        this.category.parentCid = data.catId;//菜单所属父分类ID
        this.category.catLevel = data.catLevel * 1 + 1;//菜单层级=父层级+1
        console.log('新增的菜单：', this.category);

        this.$http({
          url: this.$http.adornUrl('/product/category/save'),
          method: 'post',
          data: this.$http.adornData(this.category, false)
        }).then(({ data }) => {
          // 添加完成，刷新新的菜单
          this.getMenus();
          // 添加节点的父节点展开
          this.expandedKeys = [this.category.parentCid];
          console.log("添加节点的父节点展开:", this.expandedKeys);
          this.$message({
            type: 'success',
            message: '添加成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '添加失败!'
          });
        });;
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '取消添加'
        });
      });
    },
    // ==========================移除节点==========================
    remove(node, data) {
      console.log("node data:", node, data);
      // 打开消息确认框
      this.$confirm(`是否删除<strong><ins>${data.name}</ins></strong>?`, '提示', {
        dangerouslyUseHTMLString: true,//message 就会被当作 HTML 片段处理。
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let ids = [data.catId]; // 确定删除操作，获取当前节点id
        // 发送删除请求
        this.$http({
          url: this.$http.adornUrl('/product/category/delete/tree'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          // 删除成功提示
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
          this.getMenus(); // 删除完成，刷新新的菜单
          this.expandedKeys = [node.parent.data.catId]; // 删除节点的父节点展开
        })
      }).catch(() => {
        // 取消删除操作
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    // ==========================修改：回显节点==========================
    edit(data) {
      this.centerDialogVisible = true;//打开对话框
      console.log("edit...", data);
      // 发送请求，获取当前节点最新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(({ data }) => {
        //将获取的值赋值给表单category
        this.category.catId = data.category.catId;
        this.category.name = data.category.name;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.parentCid = data.category.parentCid;
        this.category.catLevel = data.category.catLevel;
        this.category.sort = data.category.sort;
        this.category.showStatus = data.category.showStatus;
        console.log("将获取的最新值赋值给category:", this.category);
      })

    },
    //==========================修改：提交节点========================== 
    submitData() {
      // 需要数据库只修改这4个属性,其他字段不更新，不传给API,保持其原有。
      let { catId, name, icon, productUnit } = this.category;
      console.log('修改的菜单：', { catId, name, icon, productUnit });
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data }) => {
        this.centerDialogVisible = false;//修改完成关闭对话框
        this.getMenus(); // 修改完成，刷新新的菜单
        this.expandedKeys = [this.category.parentCid]; // 添加节点的父节点展开
        console.log("修改节点的父节点展开:", this.expandedKeys);
        this.$message({
          type: 'success',
          message: '修改成功!'
        });
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '添修失败!'
        });
      });;
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
