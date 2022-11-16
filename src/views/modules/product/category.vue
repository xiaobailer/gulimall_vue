<template>
  <div>
    <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽"></el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      node-key="catId"
      show-checkbox
      v-bind:default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="data.catLevel<=2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >Append</el-button>
          <el-button type="text" size="mini" @click="edit(data)">Edit</el-button>
          <el-button
            v-if="data.child.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>

    <!-- :before-close="handleClose" -->
    <el-dialog
      :title="title"
      :visible.sync="dialogFormVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
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
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      pCid: [],
      draggable:false,
      updateNodes: [],
      maxLevel: 0,
      title: "",
      dialogType: "", //edit,add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        productUnit: "",
        icon: "",
        sort: 0
      },
      dialogFormVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "child",
        label: "name"
      }
    };
  },
  computed: {},
  watch: {},
  //方法集合
  methods: {
    //获取菜单集合【包含子类】
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        console.log("成功获取到菜单数据...", data.data);
        this.menus = data.data;
      });
    },
    handleNodeClick(data) {
      console.log(data);
    },
    // handleDrop(draggingNode, dropNode, dropType, ev) {
    //   console.log("tree drop: ", dropNode.label, dropType);

    //   //获取到当前节点的父id
    //   let pCid = 0;
    //   let sibings = null;
    //   if (dropType == "before" || dropType == "after") {
    //     pCid =
    //       dropNode.parent.data.catId == undefine
    //         ? 0
    //         : dropNode.parent.data.catId;
    //     sibings = dropNode.parent.childNodes;
    //   } else {
    //     pCid = dropNode.data.catId;
    //     sibings = dropNode.childNodes;
    //   }
    //   //当前节点拖拽后的顺序
    //   for (let i = 0; i < sibings.length; i++) {
    //     if (sibings[i].data.catId == draggingNode.data.catId) {
    //       //如果遍历的是当前正在拖拽的节点
    //       let catLevel = draggingNode.level;
    //       if (sibings[i].level != draggingNode.data.catLevel) {
    //         //当前节点的层级发生变化
    //         catLevel = sibings[i].level;

    //         //修改子节点的层级
    //         this.updateChildNodeLevel(sibings[i]);
    //       }
    //       //如果遍历的是当前正在拖拽的节点
    //       this.updateNodes.push({
    //         catId: sibings[i].data.catId,
    //         sort: i,
    //         parentCid: pCid,
    //         catLevel: catLevel
    //       });
    //     } else {
    //       this.updateNodes.push({ catId: sibings[i].data.catId, sort: i });
    //     }
    //   }
    //   console.log("updateNodes", this.updateNodes);
    //   //当前拖拽的层级
    //    //3、当前拖拽节点的最新层级

    // },
    // updateChildNodeLevel(node) {
    //   if (node.childNodes.length > 0) {
    //     for (let i = 0; i < node.childNodes.length; i++) {
    //       var cNode = node.childNodes[i].data;
    //       this.updateNodes.push( {catId:cNode.catId,catLevel:node.childNodes[i].level});
    //       this.updateChildNodeLevel(node.childNodes[i]);
    //     }
    //   }
    // },

    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单顺序等修改成功",
          type: "success"
        });
        //刷新出新的菜单
        this.getMenus();
        //设置需要默认展开的菜单
        this.expandedKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
        // this.pCid = 0;
      });
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("handleDrop: ", draggingNode, dropNode, dropType);
      //1、当前节点最新的父节点id
      let pCid = 0;
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.pCid.push(pCid);

      //2、当前拖拽节点的最新顺序，
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          //如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            //当前节点的层级发生变化
            catLevel = siblings[i].level;
            //修改他子节点的层级
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }

      //3、当前拖拽节点的最新层级
      console.log("updateNodes", this.updateNodes);
    },
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },

    allowDrop(draggingNode, dropNode, type) {
      //1、被拖动的当前节点以及所在的父节点总层数不能大于3

      //1）、被拖动的当前节点总层数
      console.log("allowDrop:", draggingNode, dropNode, type);
      //
      this.countNodeLevel(draggingNode);
      //当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;

      console.log("深度：", deep);

      //   this.maxLevel
      if (type == "inner") {
        // console.log(
        //   `this.maxLevel：${this.maxLevel}；draggingNode.data.catLevel：${draggingNode.data.catLevel}；dropNode.level：${dropNode.level}`
        // );
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    countNodeLevel(node) {
      //找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },
    edit(data) {
      this.title = "修改菜单分类";
      this.dialogFormVisible = true;
      this.dialogType = "edit";
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        console.log("查询出来的数据", data);
        this.category.catId = data.data.catId;
        this.category.name = data.data.name;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },
    append(data) {
      console.log("append", data);
      this.title = "添加分类";
      this.dialogFormVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.dialogType = "add";
    },
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    //提交修改三级分类
    editCategory() {
      console.log("category", this.category);
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data }) => {
        this.dialogFormVisible = false;
        this.$message({
          type: "success",
          message: "修改成功!"
        });
        //刷新菜单
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
    //提交添加三级分类
    addCategory() {
      console.log("category", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.dialogFormVisible = false;
        this.$message({
          type: "success",
          message: "添加成功!"
        });
        //刷新菜单
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
    //删除菜单功能
    remove(node, data) {
      var ids = [data.catId];

      this.$confirm(`此操作将永久删除${data.name}, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "删除成功!"
            });
            //刷新菜单
            this.getMenus();
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });

      console.log("remove", node, data);
    }
  },
  created() {
    this.getMenus();
  },
  mounted() {},
  beforeCreate() {},
  beforeMount() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {},
  activated() {}
};
</script>
<style scoped>
</style>