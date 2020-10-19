<template>
  <div>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="data"
      :props="defaultProps"
      show-checkbox
      :expand-on-click-node="false"
      :default-expanded-keys="exp"
      node-key="catId"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            ype="text"
            size="mini"
            @click="append(data)"
          >
            Append
          </el-button>
          <el-button type="text" size="mini" @click="edit(data)"
            >edit</el-button
          >

          <el-button
            type="text"
            size="mini"
            @click="() => remove(node, data)"
            v-if="node.childNodes.length == 0"
          >
            Delete
          </el-button>

          
        </span>
      </span>
    </el-tree>

    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
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
        <el-button type="primary" @click="addcat()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
        icon: "",
        catId: null,
      },
      title:"",
      dialogType: "",
      dialogVisible: false,
      data: [],
      exp: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  components: {},
  activated() {
    this.getDataList();
  },
  methods: {
    append(data) {
      this.dialogType = "add";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    addcat() {
      if (this.dialogType =="add") {
        this.$http({
          url: this.$http.adornUrl("/product/category/save"),
          method: "post",
          data: this.$http.adornData(this.category, false),
        }).then(({ data }) => {
          this.$message({
            message: "菜单保存成功",
            type: "success",
          });
          //关闭对话框
          this.dialogVisible = false;
          //刷新出新的菜单
          this.getDataList();
          //设置需要默认展开的菜单
          this.exp = [this.category.parentCid];
        });
      } else {
        // var {catId, name, icon, productUnit} = this.category;
        this.$http({
          url: this.$http.adornUrl("/product/category/update"),
          method: "post",
          data: this.$http.adornData(this.category, false),
        }).then(({ data }) => {
          this.$message({
            message: "菜单保存成功",
            type: "success",
          });
          //关闭对话框
          this.dialogVisible = false;
          //刷新出新的菜单
          this.getDataList();
          //设置需要默认展开的菜单
          this.exp = [this.category.parentCid];
        });
      }
    },
     batchDelete() {
        let catIds = [];
        let checkedNodes = this.$refs.menuTree.getCheckedNodes();
        for (let i = 0; i < checkedNodes.length; i++) {
          catIds.push(checkedNodes[i].catId);
        }
        this.$confirm(`是否批量删除【${catIds}】菜单?`, "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            this.$http({
              url: this.$http.adornUrl("/product/category/delete"),
              method: "post",
              data: this.$http.adornData(catIds, false)
            }).then(({data}) => {
              this.$message({
                message: "菜单批量删除成功",
                type: "success"
              });
              this.getDataList();
            });
          })
          .catch(() => {
          });
      },
    edit(data) {
      this.dialogType = "edit";
      this.title = "修改分类";
      this.dialogVisible = true;
      //发送请求获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        //请求成功
        console.log(data);
        this.category.name = data.category.name;
        this.category.catId = data.category.catId;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.parentCid = data.category.parentCid;
        this.category.catLevel = data.category.catLevel;
        this.category.sort = data.category.sort;
        this.category.showStatus = data.category.showStatus;
      });
    },

    remove(node, data) {
      var ids = [data.catId];
      this.$confirm("此操作将删除, 是否继续?", "提示", {
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
            this.$message({
              message: "删除成功",
              type: "success",
            });
            this.getDataList();
            this.exp = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
    getDataList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log(data);
        this.data = data.data;
      });
    },
  },
};
</script>
