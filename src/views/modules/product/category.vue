<template>
  <el-container>
    <el-aside width="500px">
      <div>
        <el-tree show-checkbox node-key="catId" :data="menus" :default-expanded-keys="expandedKeys"
          @node-click="handleNodeClick" :expand-on-click-node="true" :allow-drop="allowDrop"
          @node-drag-enter="handleDragEnter" @node-drop="handleDrop" :draggable="draggableSwitch" width="50%"
          ref="menuTree">
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
      </div>

    </el-aside>
    <el-container>
      <!-- <el-header>
      </el-header> -->
      <el-main>

        <el-form ref="form" label-width="80px">
          <el-form-item label="拖拽排序">
            <el-switch v-model="draggableSwitch" active-text="开启" inactive-text="关闭">
            </el-switch>
          </el-form-item>
          <el-form-item label="批量删除">
            <el-button type="danger" @click="batchDelete">提交删除</el-button>
          </el-form-item>
        </el-form>

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
      </el-main>

    </el-container>
  </el-container>
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
      parentCid: 0,
      draggableSwitch: false,
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
      sortNode: [],
      maxLevel: 0,//获取操作节点的子节点最大Level的值
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
    // ==========================点击节点==========================
    handleNodeClick(data) {
      console.log(data.catId, data.name, data.parentCid);
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
    },
    //==========================拖拽操作==========================
    // 拖拽时判定目标节点能否被放置。
    // type 参数有三种情况：'prev'、'inner' 和 'next'，分别表示放置在目标节点前、插入至目标节点和放置在目标节点后
    allowDrop(draggingNode, dropNode, type) {
      // console.log("参数:", draggingNode, dropNode, type)
      this.countNodeLevel(draggingNode.data);
      let deep = this.maxLevel - draggingNode.data.catLevel + 1; //计算拖动节点的层级
      console.log("当前节点的最大深度：", deep);
      if (type == 'inner') {
        let bool = (deep + dropNode.level) <= 3;
        // console.log(draggingNode.data.name, "插入->", dropNode.data.name, "【中】, 当前深度(", deep, ")+目标(", dropNode.level, ")<=3？", bool);
        return bool;
      } else {
        let bool = (deep + dropNode.parent.level) <= 3;
        // console.log(draggingNode.data.name, "放置->", dropNode.data.name, "【前后】, 当前深度(", deep, ")+目标父(", dropNode.parent.level, ")<=3？", bool);
        return bool;
      }
    },
    //递归：取操作节点的子节点最大Level的值
    countNodeLevel(node) {
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.children[i].catLevel;
          }
          this.countNodeLevel(node.children[i]);
        }
      }
    },
    // 拖拽进入其他节点时触发的事件
    handleDragEnter(draggingNode, dropNode, ev) {
      this.$message(`移动到：${dropNode.data.name}`);
    },
    // 拖拽成功完成时触发的事件
    // 共四个参数，依次为：被拖拽节点对应的 Node、结束拖拽时最后进入的节点、被拖拽节点的放置位置（before、after、inner）、event
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // 实时提交开启一下代码注释-1
      // this.sortNode = [];
      console.log("拖拽成功完成时触发的事件: 操作节点:", draggingNode, "->目标", dropNode, "位置:【", dropType, "】");
      console.log("拖拽成功完成时触发的事件: 操作节点名", draggingNode.data.name, "->目标名", dropNode.data.name, "位置:【", dropType, "】");
      console.log("拖拽成功完成时触发的事件: 操作节点层级", draggingNode.data.catLevel, "->目标层级", dropNode.level, "位置:【", dropType, "】");
      // 1.从dropNode获取 当前拖拽节点的最新父节点ID
      let brother = null;
      if (dropType == 'inner') {
        // 1.1.当防止位置为目标节点的内部，那么当前拖拽节点的最新父节点ID，就是目标节点的id
        this.parentCid = dropNode.data.catId;
      } else {
        // 1.2.否则就获取当前目标节点的父节点id
        this.parentCid = dropNode.data.parentCid;
      }
      console.log("最新父节点ID:", this.parentCid);

      // 2.从dropNode获取 当前拖拽节点的最新排序
      if (dropType == 'inner') {
        // 2.1.当防止位置为目标节点的内部，那么当前拖拽节点的兄弟，就是目标节点的
        // dropNode.data.children; 不能获取原始数据的子节点，应该获取拖拽完成后的子节点
        brother = dropNode.childNodes;
      } else {
        // 1.2.否则就拖拽完成后的通过父节点获取平级兄弟节点
        brother = dropNode.parent.childNodes;
      }
      console.log("最新父节点ID:", this.parentCid, "最新兄弟节点:", brother);

      // 4.在3基础上获取最新的层级
      for (let i = 0; i < brother.length; i++) {
        if (draggingNode.data.catId == brother[i].data.catId) {//3.1.遍历到自己时，给当前拖拽的节点,添加上最新的id

          console.log(draggingNode.level, "->?", brother[i].level);
          let newLevel = draggingNode.level;
          if (draggingNode.level != brother[i].level) {//拖拽级别=目标级别
            newLevel = brother[i].level;
            // 修改子分类层级
            this.updatechildNodesLevel(brother[i]);
          }
          this.sortNode.push({ catId: brother[i].data.catId, sort: i, parentCid: this.parentCid, catLevel: newLevel });  //3.2.从数组的原始对象获取ID,然后封装成新的对象，并且给排序字段赋值，放在draggableNode。
        } else {//遍历到不是自己时，层级关系不变
          this.sortNode.push({ catId: brother[i].data.catId, sort: i });  //3.2.从数组的原始对象获取ID,然后封装成新的对象，并且给排序字段赋值，放在draggableNode。
        }
      }
      console.log("最新排序的ID:", this.sortNode);

      // 实时提交开启一下代码注释-2
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.sortNode, false)
      }).then(({ data }) => {
        this.$message({
          type: 'success',
          message: '排序修改完成!'
        });
        this.getMenus(); // 删除完成，刷新新的菜单
        this.expandedKeys = [this.parentCid]; // 删除节点的父节点展开
        this.sortNode = [];
        this.maxLevel = 0//获取操作节点的子节点最大Level的值
        this.parentCid = 0;//清空要展开的节点
      });



      // 3.获取这些排好序的兄弟的id
      // for (let i = 0; i < brother.length; i++) {
      //   if (draggingNode.data.catId == brother[i].data.catId) {
      //     console.log(draggingNode.data.catId, "==", brother[i].data.catId);
      //     //3.1.遍历到自己时，给当前拖拽的节点,添加上最新的id
      //     this.sortNode.push({ catId: brother[i].data.catId, sort: i, parentCid: parentCid });
      //   } else {
      //     console.log(draggingNode.data.catId, "!=", brother[i].data.catId);
      //     //3.2.从数组的原始对象获取ID,然后封装成新的对象，并且给排序字段赋值，放在draggableNode。
      //     this.sortNode.push({ catId: brother[i].data.catId, sort: i });
      //   }
      // }
    },
    // 更新子节点的层级
    updatechildNodesLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let childCatId = node.childNodes[i].data.catId;//新位置原始数据的catId
          let childLevel = node.childNodes[i].level;//新位置的层级
          this.sortNode.push({ catId: childCatId, catLevel: childLevel });
          this.updatechildNodesLevel(node.childNodes[i]);//递归
        }
      }
    },
    //批量删除
    batchDelete() {
      // this拿到vue对象中所有组件，获取$refs中名字为menuTree的
      // getCheckedNodes：若节点可被选择（即 show-checkbox 为 true），则返回目前被选中的节点所组成的数组
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log('checkedNodes:', checkedNodes);

      // 获取所有对象的id
      let checkedIds = [];
      let checkedNames = [];
      for (let i = 0; i < checkedNodes.length; i++) {
        const node = checkedNodes[i];
        checkedIds.push(node.catId);
        checkedNames.push(node.name);
      }
      console.log('checkedIds:', checkedIds);
      this.doBatchDelete(checkedIds,checkedNames);
    },
    doBatchDelete(checkedIds,checkedNames) {
      this.$confirm(`是否删除<strong><ins>${checkedNames}</ins></strong>?`, '提示', {
        dangerouslyUseHTMLString: true,//message 就会被当作 HTML 片段处理。
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 发送删除请求
        this.$http({
          url: this.$http.adornUrl('/product/category/delete/tree'),
          method: 'post',
          data: this.$http.adornData(checkedIds, false)
        }).then(({ data }) => {
          // 删除成功提示
          this.$message({
            type: 'success',
            message: '批量删除成功!'
          });
          this.getMenus(); // 删除完成，刷新新的菜单
        })
      }).catch(() => {
        // 取消删除操作
        this.$message({
          type: 'info',
          message: '已取消批量删除'
        });
      });
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
