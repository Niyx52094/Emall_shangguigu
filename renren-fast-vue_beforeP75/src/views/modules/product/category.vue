<!--  -->
<template>
  <div>
    <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽"></el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level<3" type="text" size="mini" @click="() => append(data)">Append</el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>

          <el-button type="text" size="mini" @click="()=>edit(data)">Edit</el-button>
        </span>
      </span>
    </el-tree>
    <!-- 新增框和修改框-->
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>

    <!-- <el-dialog title="提示" :visible.sync="dialogVisible4Update" width="30%">
      <el-form :model="category">
        <el-form-item label="修改名称">
          <el-input v-model="category.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="修改菜单父级">
          <el-input v-model="category.parentCid" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="修改当前菜单">
          <el-input v-model="category.catLevel" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible4Update = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>  修改框-->
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      pCid: [],
      draggable: false,
      updateNodes: [],
      maxlevel: 0,
      title: "",
      dialogType: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: ""
      },
      dialogVisible: false,
      expandedKey: [],
      menus: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    getMenus() {
      this.reset();
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get" //发送什么请求，
      }).then(({ data }) => {
        console.log("成功获取到菜单数据.....", data.data);
        this.menus = data.data;
      });
    },
    reset() {
      this.updateNodes = [];
      this.maxlevel = 0;
      //this.pCid = [];
    },

    //批量保存
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        //刷新数据
        this.getMenus();
        this.expandedKey = this.pCid;
        //发送成功信息
        this.$message({
          type: "success",
          message: "菜单顺序修改成功!"
        });
      });
    },
    //拖拽节点成功后触发的事件。就是要更新数据库
    handleDrop(draggingNode, dropNode, dropType, ev) {
      let pCid = 0;
      let siblings = null;
      if (dropType == "inner") {
        //改父节点为进入节点的节点数，
        pCid = dropNode.data.catId;
        //找兄弟节点
        siblings = dropNode.childNodes;
      } else {
        //赋值为before和after那个dropNode的父节点
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        //找兄弟节点
        siblings = dropNode.parent.childNodes;
      }

      //排序
      for (let i = 0; i < siblings.length; i++) {
        //如果遍历到拖拽的节点

        if (siblings[i].data.catId == draggingNode.data.catId) {
          //如果层级改变则需要进行更新
          let catLevel = draggingNode.level;
          if (catLevel != siblings[i].level) {
            //当前层级发生变化：
            catLevel = siblings[i].level;
            this.childLevelUpdate(siblings[i]);
          }
          //排好序放到updateNodes,还可能需要改父节点
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel //放入改变的层级
          });
        } else {
          //排好序放到updateNodes
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      this.pCid.push(pCid);
      console.log("pCid: ", pCid);
      console.log("updatedNodes: ", this.updateNodes);
      console.log("handleDrop: ", draggingNode, dropNode, dropType);
    },
    //每个children的层级都改变
    childLevelUpdate(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          //放入修改的层级并识别。
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level
          });
          this.childLevelUpdate(node.childNodes[i]);
        }
      }
    },
    //拖动节点
    allowDrop(draggingNode, dropNode, type) {
      //计算被拖动的当前节点的总层数
      this.countDepth(draggingNode);
      var draggingDepth = Math.abs(this.maxlevel - draggingNode.level + 1);
      if (type == "inner") {
        //被拖动的当前节点的总层数加上目标位置节点的父层数不能大于3
        return draggingDepth + dropNode.level <= 3;
      } else {
        //如果不是放入。那么就是和父节点的level相加小于等于3
        return draggingDepth + dropNode.parent.level <= 3;
      }
      //console.log("allowDrop", draggingNode, dropNode, type);
      //return false;
    },
    //计算深度的方法
    countDepth(node) {
      if (node.childNodes.length > 0 && node.childNodes != null) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxlevel) {
            this.maxlevel = node.childNodes[i].level;
          }
          this.countDepth(node.childNodes[i]);
        }
      }
    },

    submitData() {
      if (this.dialogType == "edit") {
        this.updateCategory();
      } else if (this.dialogType == "add") {
        this.addCategory();
      }
    },
    edit(data) {
      this.title = "添加分类";
      this.dialogType = "edit";
      this.dialogVisible = true;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        console.log("要修改的数据：", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });

      //下面时回显的当时的数据，但有可能会被人改过了，所以正确的做法是发送请求，获得当前的数据。
      // this.category.name = data.name;
      // this.category.catId = data.catId;
      // this.category.icon=data.icon;
      // this.category.productUnit=data.productUnit;
    },

    //确认修改
    updateCategory() {
      var { catId, name, icon, productUnit } = this.category;
      var data = {
        catId: catId,
        name: name,
        icon: icon,
        productUnit: productUnit
      };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false)
      }).then(({ data }) => {
        //关闭对话框
        this.dialogVisible = false;
        //刷新数据
        this.getMenus();
        //发送成功信息
        this.$message({
          type: "success",
          message: "修改成功!"
        });
        //展开父菜单
        this.expandedKey = [this.category.parentCid];
      });
    },

    append(data) {
      this.category = {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: ""
      };
      this.title = "添加分类";
      this.dialogType = "add";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1; //防止取回来的是字符串所以乘以1。变为数字

      console.log("append", data);
    },
    //添加三级分类
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        //关闭对话框
        this.dialogVisible = false;
        this.getMenus();
        //展开父菜单
        this.expandedKey = [this.category.parentCid];

        //发送成功信息
        this.$message({
          type: "success",
          message: "新增成功!"
        });

        console.log("添加的数据是", this.category);
      });
    },

    batchDelete() {
      let canIds = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("被删除的元素", checkedNodes);
      for (let i = 0; i < checkedNodes.length; i++) {
        //把Id封装成数组。
        canIds.push(checkedNodes[i].catId);
      }

      //弹出确认提示框
      this.$confirm(`是否批量删除选中【${canIds}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(canIds, false)
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "批量删除成功!"
            });
            this.getMenus();
            //展开父菜单
            //this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },

    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单`, "提示", {
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

            this.getMenus();

            //展开父菜单
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
  activated() {} //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>