<template>
  <div>
    <!-- el-breadcrumb -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- card -->
    <el-card>
      <!-- 添加角色button -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区 -->
      <el-table :data="roleList" border stripe>
        <!-- type="expand" 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom',i1 === 0 ? 'bdtop' : '','vcenter']" v-for="(item1 , i1) in scope.row.children" 
                    :key="item1.id" :gutter="24">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二、三级权限 -->
              <el-col :span="19">
                <!-- for嵌套渲染二级 -->
                <el-row :class="[i2 === 0 ? '' :'bdtop','vcenter']" v-for="(item2,i2) in item1.children"
                        :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3,i3) in item2.children"
                            :key="item3.id">
                    {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- index索引列 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditRoleDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色的对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体 -->
      <el-form
        :model="addRole"
        :rules="RoleRules"
        ref="addRoleRef"
        label-width="70px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRole.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRole.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles"> 确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色的对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 内容主体 -->
      <el-form
        :model="editRole"
        :rules="RoleRules"
        ref="editRoleRef"
        label-width="70px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRole.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRole.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo"> 确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限"
               :visible.sync="setRightDialogVisible"
               width="50%"
               @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightsList" :props="treeProps" 
               show-checkbox
               node-key="id"
               default-expand-all
               :default-checked-keys="defKeys"
               ref="treeRef">
      </el-tree>
      <span slot="footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],//角色列表
      addRole:{
        roleName:'',
        roleDesc:''
      },//添加角色
      addDialogVisible: false,//dialog对话框是否显示
      RoleRules:{//添加，修改角色校验
        roleName: [
          {required: true, message:"请输入角色名称" , trigger: 'blur'}
        ],
        roleDesc: []
      },
      editDialogVisible: false,//是否显示编辑角色dialog
      editRole: {//查询到的role信息
      
      },
      setRightDialogVisible: false, //控制分配权限dialog是否隐藏
      rightsList: [] , //获取到的权限数据
      treeProps:{//树形控件的属性绑定对象
        label:'authName', //显示的是authName
        children: 'children'
      },
      defKeys:[],//默认选中的节点id
      roleId: '',//即将分配角色权限的roleid
    };
  },
  created() {
    this.getRoleList();
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200)
        return this.$message.error("获取角色列表信息失败");
      this.roleList = res.data;
    },
    //添加角色
    addRoles(){
      this.$refs.addRoleRef.validate(async valid => {
        console.log(valid);
        if(!valid) return 
        const {data:res} = await this.$http.post('roles',this.addRole)
        if(res.meta.status !=201) return this.$message.error('添加角色失败')
        this.$message.success('添加角色成功')
        this.addDialogVisible = false
        this.getRoleList()
      })
    },
    // 监听添加角色对话框的关闭事件
    addDialogClosed() {
      this.$refs.addRoleRef.resetFields()
    },
    //编辑角色
    async showEditRoleDialog(id) {
      const {data:res} =await this.$http.get('roles/' + id)
      if(res.meta.status !== 200) return this.$message.error("查询角色信息失败")
      this.editDialogVisible = true
      this.editRole = res.data
    },
    //编辑角色提交
    editRoleInfo() {
      //预校验
      this.$refs.editRoleRef.validate(async valid => {
        if(!valid) return
        console.log(this.editRole.id);
        const {data:res} = await this.$http.put("roles/" + this.editRole.roleId,this.editRole) 
        console.log(res);
        if(res.meta.status !== 200) return this.$message.error('编辑角色信息失败')
        this.getRoleList()
        this.$message.success('编辑角色信息成功')
        this.editDialogVisible = false
      })
    },
    editDialogClosed() {
      this.editDialogVisible = false
    },
    //删除角色
    async removeRoleById(id){
     const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)  //err => {return err}
        //确定删除confirm
        //取消删除cancel
        if(confirmResult !=='confirm') {
          return this.$message.info('已取消删除操作')
        }
        const {data:res} = await this.$http.delete('roles/'+ id)
        if(res.meta.status !== 200) return this.$message.error('删除角色失败')
        this.$message.success('删除角色成功')
        this.getRoleList()

    },
    //分配权限对话框=========================================================
    async showSetRightDialog(role) {
      this.roleId = role.id
      //获取权限信息
      const {data:res} = await this.$http.get('rights/tree')
      if(res.meta.status !== 200) return this.$message.error('获取权限信息失败')
      this.rightsList = res.data
      //三级权限id添加到defKeys[]中 | 递归函数
      this.getLeafKeys(role,this.defKeys)
      this.setRightDialogVisible = true
    },
    //递归函数，获取角色的三级权限id，并保存到defKeys[]
    getLeafKeys(node, arr) {
      if(!node.children) {//node不包含children，（即三级）直接拿id
        return arr.push(node.id)
      }
      node.children.forEach(item => { // forEach() children的每个都执行forEach中的函数
        this.getLeafKeys(item,arr)
      })
    },
    //关闭分配权限对话框，清除defKeys数据
    setRightDialogClosed() {
      this.defKeys = []
    },
    //分配权限
    async allotRights() {
      const keys =[
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ] 
      const idStr = keys.join(',') //join() 字符串拼接
      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids: idStr})
      if(res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('分配权限成功')
      this.getRoleList()
      this.setRightDialogVisible = false
    }
    
  },
};
</script>

<style scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>