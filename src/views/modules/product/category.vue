<template>
  <div>
    <!-- 拖拽功能 -->
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <!-- element-ui Tree树形菜单 -->
    <!-- 
    show-checkbox： 节点是否可被选择 显示选择框
    :expand-on-click-node="false" 如果为 false，则只有点箭头图标的时候才会展开或者收
    node-key： 每个树节点用来作为唯一标识的属性，整棵树应该是唯一的
   -->
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      ref="menuTree"
      @node-drop="handleDrop"
    >
      >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <!-- 只有父菜单才显示append-->
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <!-- 只用子菜单才显示delete -->
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
          <!-- 任意菜单可以进行修改 -->
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <!-- <span>这是一段信息</span> -->
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
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
  directives: {},
  data() {
    return {
      pCid: [], // 拖拽中使用的全局变量 -- 需要展开的父节点
      draggable: false, // 开启/固安必拖拽功能
      updateNodes: [], // 拖拽更新的节点
      maxLevel: 0, // 最大节点层级，拖拽效果
      dialogTitle: "", // 对话框标题
      dialogType: "", // 复用对话框 edit append
      menus: [], // 菜单
      expandedKey: [], // 刷新展开菜单id
      dialogVisible: false, // 对话框显示
      category: {
        catId: null,
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
      }, // 表单空对象
      defaultProps: {
        children: "children",
        label: "name", // 显示的标签
      },
    };
  },
  mounted() {},
  methods: {
    // 添加菜单
    append(data) {
      // 打开对话框
      this.dialogTitle = "添加分类";
      this.dialogVisible = true;
      this.dialogType = "append";
      // 计算相关属性值
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;

      // 数据回显清空\
      this.category.catId = null;
      this.category.name = "";
      this.category.productUnit = "";
      this.category.icon = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },
    // 添加三级分类
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        // 成功提示消息
        this.$message({
          type: "success",
          message: "菜单添加成功！",
        });
        // 关闭对话框
        this.dialogVisible = false;
        // 刷新菜单数据
        this.getMenus();
        // 展开默认菜单
        this.expandedKey = [this.category.parentCid];
      });
      // console.log("提交的分类数据：", this.category);
    },
    // 刪除菜單
    remove(node, data) {
      // console.log("remove", node, data);
      var ids = [data.catId];
      // 删除前弹框提示
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            // 成功提示消息
            this.$message({
              type: "success",
              message: "菜单删除成功！",
            });
            //console.log("删除成功....");
            this.getMenus(); // 刷新
            // 显示删除后的默认节点
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    // 修改
    edit(data) {
      // console.log("要修改的数据:", data);
      // 显示对话框
      this.dialogTitle = "修改分类";
      this.dialogVisible = true;
      this.dialogType = "edit";
      // 发送请求获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        // 请求成功 -- 回显最新的数据
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.productUnit = data.data.productUnit;
        this.category.icon = data.data.icon;
        this.category.parentCid = data.data.parentCid;
        // 数据回显 --- 发送category,要将category中属性都回显
        // this.category.catLevel = data.data.catLevel;
        // this.category.showStatus = data.data.showStatus;
        // this.category.sort = data.data.sort;
      });
    },
    // 修改三级分类数据
    editCategory() {
      // 点击确定发送修改请求 -- 不更新的数据不发,否则默认值会覆盖数据库中的数据
      // 解构
      var { catId, name, icon, productUnit } = this.category;
      // Key-Value KV相同可以省略K：V
      var data = {
        catId: catId,
        name: name,
        icon: icon,
        productUnit: productUnit,
      };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        // data: this.$http.adornData(this.category, false),
        data: this.$http.adornData(data, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功",
          type: "success",
        });
        // 关闭对话框
        this.dialogVisible = false;
        // 刷新菜单
        this.getMenus();
        // 设置需要展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },
    // 提交添加或修改数据
    submitData() {
      if (this.dialogType === "append") {
        this.addCategory();
      }
      if (this.dialogType === "edit") {
        this.editCategory();
      }
    },
    // 拖拽效果
    allowDrop(draggingNode, dropNode, type) {
      // console.log(draggingNode, dropNode, type);
      // 当前节点，目标节点，哪个位置
      // 判断条件：被拖动的当前节点和所在父节点总层数不能大于3
      // 统计被拖动节点的总层数 maxLevel
      this.countNodeLevel(draggingNode);
      // 最大深度 -- 拖拽节点的层数
      let deep = this.maxLevel - draggingNode.data.catLevel + 1;
      this.maxLevel = 0; // 重置为0
      if (type == "inner") {
        // 拖到内部
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    countNodeLevel(node) {
      this.maxLevel = node.level; // 解决四级拖拽Bug
      // 找到所有节点，求出最大深度 -- 有子节点，递归查询
      if (node.childNodes != null && node.childNodes.length > 0) {
        // 遍历子节点
        for (let i = 0; i < node.children.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = this.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },
    // 处理拖拽成功事件
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // console.log("tree drop: ", dropNode.label, dropType);
      // 1、当前节点最新的父节点id
      let siblings = null;
      let pCid = 0;
      if (dropType == "before" || dropType == "after") {
        // 拖拽到的节点没有子级菜单
        // 如果称为根级菜单，则父id为null
        pCid =
          dropNode.parent.data.catId == undefined
            ? null
            : dropNode.parent.data.catId;
        // 兄弟节点收集后重新排序
        siblings = dropNode.parent.childNodes;
      } else {
        // 叶子菜单
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }

      this.pCid.push(pCid);
      // 2、当前拖拽节点的最新排序
      for (let i = 0; i < siblings.length; i++) {
        // 整理兄弟节点顺序后包装为对象发给后端
        // 拖拽节点可能导致父子关系变化
        if (siblings[i].data.catId == draggingNode.data.catId) {
          // 如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            // 当前节点的层级发生变化
            catLevel = siblings[i].level;
            // 修改子节点的层级
            this.updateChildNodeLevel(siblings[i]);
          }
          // 需要更改父id
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      // 3、当前拖拽节点的最新层级
    },
    // 修改子节点的层级
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.length; i++) {
          var cNode = node.childNodes[i].data;
          // 更新层级id和level
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          // 递归子节点
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    // 批量保存
    batchSave() {
      // 发送给后端
      this.$http({
        url: this.$http.adornUrl("product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单层级拖拽成功！",
          type: success,
        });
        // 刷新菜单
        this.getMenus();
        // 设置默认展开菜单，上一次拖拽的菜单
        this.expandedKey = this.pCid;
        // 清空历史数据
        this.updateNodes = [];
        this.maxLevel = 0;
        // this.pCid = 0;
      });
    },
    // 批量删除
    batchDelete() {
      // (leafOnly, includeHalfChecked) 接收两个 boolean 类型的参数，1. 是否只是叶子节点，默认值为 `false` 2. 是否包含半选节点，默认值为 `false`
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      let catIds = [];
      // console.log("被选中的元素:",this.$refs.menuTree.getCheckedNode())
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
      }
      // 删除前弹框提示
      this.$confirm(`是否批量删除选中菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          })
            .then(({ data }) => {
              this.$message({
                type: "success",
                message: "批量删除成功",
              });
              // 刷新菜单
              this.getMenus();
            })
            .catch(() => {});
        })
        .catch(() => {});
    },
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
  },
  created() {
    this.getMenus();
  },
};
</script>

<style scoped>
</style>