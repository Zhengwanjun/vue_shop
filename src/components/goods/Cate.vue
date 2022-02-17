<template>
  <div>
    <!-- el-breadcrumb -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- el-card -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!-- table 第三方插件 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        class="treeTable"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            type="success"
            size="mini"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            circle
            @click="showEditCateDialog(scope.row.cat_id)"
          ></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            circle
            @click="removeCateById(scope.row.cat_id)"
          ></el-button>
        </template>
      </tree-table>
      <!-- page -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类的dialog -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
      ref="addCateFormRef"
    >
      <!-- 内容主体 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="70px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- 级联选择 options指定数据源 props用来指定配置对象 -->
          <el-cascader
            expand-trigger="hover"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChange"
            v-model="selectedKeys"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate"> 确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类的dialog -->
    <el-dialog
      title="修改分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="editCateDialogClosed"
    >
      <!-- 内容主体 -->
      <el-form
        :model="editCateForm"
        :rules="editCateFormRules"
        ref="editCateFormRef"
        label-width="70px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>

      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate"> 确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cateList: [], //商品分类的数据列表
      queryInfo: {
        //查询条件
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      total: 0, //总数据条数
      columns: [
        //为table指定列的定义
        {
          label: "分类名称", //列名
          prop: "cat_name", //列数据
        },
        {
          label: "是否有效",
          type: "template", //使用自定义模板列
          template: "isok", //当前这列使用的模板名称  slot="isok"
        },
        {
          label: "排序",
          type: "template", //使用自定义模板列
          template: "order", //当前这列使用的模板名称  slot="isok"
        },
        {
          label: "操作",
          type: "template", //使用自定义模板列
          template: "opt", //当前这列使用的模板名称  slot="isok"
        },
      ],
      addCateDialogVisible: false, //控制添加分类对话框的显隐
      addCateForm: {
        cat_name: "", //将要添加的分类名称
        cat_pid: 0, //父级分类的id
        cat_level: 0, //分类的等级，默认添加一级分类
      }, //添加分类表单数据
      addCateFormRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
        ],
      },
      parentCateList: [], //父级分类列表数据
      cascaderProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      selectedKeys: [], //选中的父级分类的id数组
      editCateDialogVisible: false, //编辑分类的显隐
      editCateForm: {
        //将要编辑的分类
        cat_id: 0,
        cat_name: "",
        cat_pid: 0,
        cat_level: 0,
      },
      editCateFormRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    //获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200)
        return this.$message.error("获取商品分类列表失败");
      this.cateList = res.data.result;
      this.total = res.data.total;
    },
    //监听 pagesize 改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    //监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    //添加分类
    showAddCateDialog() {
      //获取父级分类列表
      this.getParentCateList();
      this.addCateDialogVisible = true;
    },
    //获取父级分类数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 },
      });
      if (res.meta.status !== 200)
        return this.$message.error("获取父级分类失败");
      this.parentCateList = res.data;

      console.log(this.parentCateList);
    },
    //记录分类选项发生变化
    parentCateChange() {
      console.log(this.selectedKeys);
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_level = this.selectedKeys.length;
        this.addCateForm.cat_pid =
          this.selectedKeys[this.selectedKeys.length - 1];
        return;
      } else {
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      }
    },
    //点击确定添加分类
    addCate() {
      this.$refs.addCateFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          "categories",
          this.addCateForm
        );
        if (res.meta.status !== 201) {
          this.$message.error("添加分类失败");
        }
        this.$message.success("添加分类成功");
        this.addCateForm = res.data;
        this.getCateList();
        this.addCateDialogVisible = false;
      });
    },
    //监听添加分类关闭
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields();
      this.selectedKeys = [];
      this.addCateForm.cat_pid = 0;
      this.addCateForm.cat_level = 0;
    },
    //编辑分类
    async showEditCateDialog(id) {
      const { data: res } = await this.$http.get("categories/" + id);
      if (res.meta.status !== 200) return;
      this.editCateForm = res.data;
      this.editCateDialogVisible = true;
    },
    //确定编辑分类
    editCate() {
      //预校验
      this.$refs.editCateFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          "categories/" + this.editCateForm.cat_id,
          { cat_name: this.editCateForm.cat_name }
        );
        console.log(res.data);
        if (res.meta.status !== 200) {
          return this.$message.error("编辑分类失败");
        }
        console.log(res.data);
        this.$message.success("编辑分类成功");
        this.editCateForm = res.data;
        this.getCateList();
        this.editCateDialogVisible = false;
      });
    },
    //监听编辑分类对话框关闭
    editCateDialogClosed() {
      this.editCateForm = {};
    },
    //删除分类
    removeCateById(id){
      this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async () => {
          //网络请求
          const {data:res} = await this.$http.delete('categories/' + id)
          if(res.meta.status !== 200) return
          this.getCateList()
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });
    }
  },
};
</script>

<style scoped>
.treeTable {
  margin-top: 15px;
}
</style>