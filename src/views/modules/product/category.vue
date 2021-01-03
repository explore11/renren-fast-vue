<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      show-checkbox
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="expandKeys"
      draggable
      :allow-drop="allowDrop"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            新增
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
            >删除</el-button
          >
          <el-button type="text" size="mini" @click="() => edit(node, data)"
            >修改</el-button
          >
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="菜单名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标地址">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
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
  name: "category",
  components: {},
  data() {
    return {
      maxLevel: 1,
      title: "",
      editCategoryType: "",
      category: {
        catId: null,
        name: "",
        parentCid: 0,
        catLevel: 1,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
        productCount: 0
      },
      dialogVisible: false,
      menus: [],
      expandKeys: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    // 可拖拽
    allowDrop(draggingNode, dropNode, type) {
      console.log("可拖拽", draggingNode, dropNode, type);
      // 计算当前节点的最大深度
      this.getNodeDeep(draggingNode.data);
      console.log("计算当前节点的最大深度", this.maxLevel);
      // 计算要增加的深度
      let deep = this.maxLevel - draggingNode.level + 1;
      console.log("深度", deep);
      // 根据类型判断
      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    // 计算当前节点的最大深度
    getNodeDeep(node) {
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.children[i].catLevel;
          }
          this.getNodeDeep(node.children[i]);
        }
      }
    },
    // 提交方式
    submitData() {
      if (this.editCategoryType == "add") {
        // 添加分类
        this.addCategory();
      }
      if (this.editCategoryType == "edit") {
        // 添加分类
        this.editCategory();
      }
    },

    // 获取分类
    getDataListTree() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        // {data}  结构函数  返回的数据中  取data字段
        this.menus = data.categoryEntityList;
      });
    },
    // 新增
    append(data) {
      // 清楚上次新增后的数据
      this.category.name = "";
      // 设置类型
      this.editCategoryType = "add";
      console.log("append", data);
      this.dialogVisible = true;
      this.title = "修改分类";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      //初始化
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.catId = null;
      this.category.showStatus = 1;
      this.category.sort = 0;
      this.category.productCount = 0;
    },
    // 修改
    edit(node, data) {
      console.log("修改", data);
      // 设置类型
      this.editCategoryType = "edit";
      this.dialogVisible = true;
      this.title = "修改分类";

      // 数据回显
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        params: this.$http.adornParams({})
      }).then(({ data }) => {
        console.log("数据回显", data);
        this.category.name = data.category.name;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.catId = data.category.catId;
        this.category.parentCid = data.category.parentCid;
      });
    },
    // 修改分类
    editCategory() {
      var { name, icon, productUnit, catId } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ name, icon, productUnit, catId }, false)
      }).then(({ data }) => {
        this.$message({
          message: "分类修改成功",
          type: "success"
        });
        // 关闭菜单弹窗
        this.dialogVisible = false;
        // 刷新数据
        this.getDataListTree();
        //设置默认展开的节点
        this.expandKeys = [this.category.parentCid];
      });
    },
    // 添加分类
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          message: "分类新增成功",
          type: "success"
        });
        // 关闭菜单弹窗
        this.dialogVisible = false;
        // 刷新数据
        this.getDataListTree();
        //设置默认展开的节点
        this.expandKeys = [this.category.parentCid];
      });
    },
    //删除
    remove(node, data) {
      console.log(node);
      // 数据
      var ids = [data.catId];
      this.$confirm(`此操作将永久删除${data.name}菜单, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          // 确认删除 发出请求
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              message: "菜单删除成功",
              type: "success"
            });
            // 刷新数据
            this.getDataListTree();
            //设置默认展开的节点
            this.expandKeys = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  },

  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    //调用接口
    this.getDataListTree();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {} //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>

<style lang="scss" scoped>
</style>