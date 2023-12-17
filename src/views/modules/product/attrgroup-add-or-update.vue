<template>
  <el-dialog :title="!dataForm.attrGroupId ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible"
    width="30%" @closed="dialogClosed">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
      label-width="100px">
      <el-form-item label="组名" prop="attrGroupName">
        <el-input v-model="dataForm.attrGroupName" placeholder="组名" size="small" style="width:95%"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model="dataForm.sort" placeholder="排序" size="small" style="width:95%"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="descript">
        <el-input v-model="dataForm.descript" placeholder="描述" size="small" style="width:95%"></el-input>
      </el-form-item>
      <el-form-item label="组图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="组图标" size="small" style="width:95%"></el-input>
      </el-form-item>
      <el-form-item label="所属分类" prop="catelogId">
        <!-- filterable可搜索 -->
        <el-cascader v-model="dataForm.catelogPath" :options="categorys" :props="props" filterable size="small"
          style="width:95%" placeholder="试试搜索分类..."></el-cascader>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false" size="small">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()" size="small">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      props: {
        value: "catId", // value	指定选项的值为选项对象的某个属性值	string	—	'value'
        label: "name",  // label	指定选项标签为选项对象的某个属性值	string	—	'label'
        // children: "children",  // children	指定选项的子选项为选项对象的某个属性值
        expandTrigger: 'hover'
      },
      categorys: [],
      visible: false,
      dataForm: {
        attrGroupId: 0,
        attrGroupName: '',
        sort: '',
        descript: '',
        icon: '',
        catelogPath: [],//绑定下拉框数据
        catelogId: 0//提交下拉框数据
      },
      dataRule: {
        attrGroupName: [
          { required: true, message: '组名不能为空', trigger: 'blur' }
        ],
        sort: [
          { required: true, message: '排序不能为空', trigger: 'blur' }
        ],
        descript: [
          { required: true, message: '描述不能为空', trigger: 'blur' }
        ],
        icon: [
          { required: true, message: '组图标不能为空', trigger: 'blur' }
        ],
        catelogId: [
          { required: true, message: '所属分类id不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    dialogClosed() {
      this.dataForm.catelogPath = [];
    },
    // ==================下拉框==========================
    getCategorys() {
      // 发送菜单树状结构请求
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(({ data }) => {
        //将获取的值赋值给data中的menus
        this.categorys = data.data;
      })
    },
    init(id) {
      this.dataForm.attrGroupId = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.attrGroupId) {
          this.$http({
            url: this.$http.adornUrl(`/product/attrgroup/info/${this.dataForm.attrGroupId}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              console.log("data:", data);
              this.dataForm.attrGroupName = data.attrGroup.attrGroupName
              this.dataForm.sort = data.attrGroup.sort
              this.dataForm.descript = data.attrGroup.descript
              this.dataForm.icon = data.attrGroup.icon
              this.dataForm.catelogId = data.attrGroup.catelogId
              this.dataForm.catelogPath = data.attrGroup.catelogPath
            }
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/product/attrgroup/${!this.dataForm.attrGroupId ? 'save' : 'update'}`),
            method: 'post',
            data: this.$http.adornData({
              'attrGroupId': this.dataForm.attrGroupId || undefined,
              'attrGroupName': this.dataForm.attrGroupName,
              'sort': this.dataForm.sort,
              'descript': this.dataForm.descript,
              'icon': this.dataForm.icon,
              'catelogId': this.dataForm.catelogPath[this.dataForm.catelogPath.length - 1]//只提交数组最后一个值
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  // 提交成功 刷新父组件
                  this.$emit('refreshDataList')
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }
      })
    }
  },
  // 组件创建出来就执行此方法
  created() {
    this.getCategorys();
  }
}
</script>
