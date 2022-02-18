<template>
  <div>
    <!-- el-breadcrumb -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- card -->
    <el-card>
      <!-- 提示框 -->
      <el-alert
        title="注意：只允许为第三级分类设置相关参数"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <span>选择商品分类：</span>
        <!-- 级联选择 -->
        <el-cascader
          expand-trigger="hover"
          v-model="selectedCateKeys"
          :options="cateList"
          :props="cateProps"
          @change="handleChange"
        ></el-cascader>
      </el-row>
      <!-- tab -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加动态参数面板 -->
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisable"
            @click="adddialogVisible = true"
            >添加参数</el-button
          >
          <!-- table -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag -->
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClosed(i,scope.row)"
                >
                  {{ item }}
                </el-tag>
                <!-- 添加 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!--  -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  circle
                  @click="showEditDialog(scope.row.attr_id)"
                ></el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  circle
                  @click="removeParams(scope.row.attr_id)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <el-tab-pane label="静态属性" name="only">
          <!-- 添加静态属性面板 -->
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisable"
            @click="adddialogVisible = true"
            >添加属性</el-button
          >
          <!-- table -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag -->
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClosed(i,scope.row)"
                >
                  {{ item }}
                </el-tag>
                <!-- 添加 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!--  -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column
              label="属性名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  circle
                  @click="showEditDialog(scope.row.attr_id)"
                ></el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  circle
                  @click="removeParams(scope.row.attr_id)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="adddialogVisible"
      width="30%"
    >
      <!--  -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        @close="addDialogClosed"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="adddialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定 </el-button>
      </span>
    </el-dialog>
    <!-- 修改Dialog -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editdialogVisible"
      width="30%"
    >
      <!--  -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        @close="editDialogClosed"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定 </el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cateList: [], //商品分类列表
      cateProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      }, //级联选择框的配置对象
      selectedCateKeys: [], //级联选择框双向绑定的
      activeName: "many", //被激活的tab名称,默认first的
      manyTableData: [], //动态参数数据
      onlyTableData: [], //静态属性数据
      adddialogVisible: false, //控制添加对话框的显隐
      addForm: {
        //添加参数的表单数据对象
        attr_name: "",
      },
      addFormRules: {
        //添加参数的表单数据验证规则对象
        attr_name: [
          { required: true, message: "请输入参数名称", trigger: "blur" },
        ],
      },
      editdialogVisible: false,
      editForm: {
        attr_name: "",
      },
      editFormRules: {
        attr_name: [
          { required: true, message: "请输入参数名称", trigger: "blur" },
        ],
      },

    };
  },
  //终于用到computed了!!
  computed: {
    //如果按钮需要被禁用则return true
    isBtnDisable() {
      if (this.selectedCateKeys.length !== 3) {
        return true;
      }
      return false;
    },
    //当前选中的三级分类的id
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2];
      }
      return null;
    },
    //动态计算标题的文本
    titleText() {
      if (this.activeName === "many") {
        return "动态参数";
      }
      return "静态属性";
    },
  },
  created() {
    this.getCateList();
  },
  methods: {
    //获取商品分类信息
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200)
        return this.$message.error("获取商品分类失败");
      this.cateList = res.data;
    },
    //级联选择框选中项变化,会触发此函数
    handleChange() {
      this.getParamsData();
    },
    //Tab点击事件监听
    handleTabClick() {
      this.getParamsData();
    },
    //获取参数的列表数据
    async getParamsData() {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = [];
        this.manyTableData = []
        this.onlyTableData = []
        return;
      }
      //发起请求,根据所选分类的id和当前所处的面板获取到当前三级分类的参数项
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数列表失败");
      }

      res.data.forEach((item) => {
        item.attr_vals = item.attr_vals
          ? (item.attr_vals = item.attr_vals.split(" "))
          : [];
          //控制文本框的显隐
          item.inputVisible = false
          //文本框中输入的值
          item.inputValue = ''
      });
      if (this.activeName === "many") {
        this.manyTableData = res.data;
      } else {
        this.onlyTableData = res.data;
      }
    },
    //添加参数对话框关闭
    addDialogClosed() {
      this.addForm = {};
      this.adddialogVisible = false;
    },
    //点击按钮添加参数
    addParams() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加参数失败");
        }
        this.$message.success("添加参数成功");

        this.getParamsData();
        this.adddialogVisible = false;
        this.$refs.editFormRef.resetFields();
      });
    },
    //点击按钮展示修改对话框
    async showEditDialog(attr_id) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`,
        {
          params: { attr_sel: this.activeName },
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数失败");
      }
      this.editForm = res.data;
      this.editdialogVisible = true;
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    editParams() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("修改参数失败");
        }
        this.$message.success("修改参数成功");
        this.getParamsData();
        this.editdialogVisible = false;
      });
    },
    //删除
    removeParams(id) {
      this.$confirm("此操作将永久删除该参数, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          //网络请求
          const { data: res } = await this.$http.delete(
            `categories/${this.cateId}/attributes/${id}`
          );
          if (res.meta.status !== 200) return;
          this.getParamsData();
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    //文本框失去焦点或者enter
    async handleInputConfirm(row) {
      if(row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      //输入的合法
      this.saveAttrVals(row)
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false

    },
    async saveAttrVals(row) {
      
      //请求
      const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
        attr_name: row.attr_name,
        attr_sel:row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if(res.meta.status !== 200) return this.$message.error('修改参数失败')

      this.$message.success('修改参数成功')
    },
    //点击按钮，展示文本输入框
    showInput(row) {
       
      row.inputVisible = true
      //$nextTick 当页面上元素被重新渲染之后，才会指定回调函数中的代码
      this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
    },
    handleClosed(i,row) {
      row.attr_vals.splice(i,1)
      this.saveAttrVals(row)
    }
  },
};
</script>

<style scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 5px;
}
.input-new-tag {
  width: 120px
}
</style>