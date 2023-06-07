<template>
  <div>
    <el-card class="box-card">

      <el-row>
        <el-col :span="20">
          <el-input v-model="searchModel.username" placeholder="用户名" clearable />
          <el-input v-model="searchModel.phone" placeholder="电话" clearable />
          <el-button type="primary" round icon="el-icon-search" @click="getUserList">查询</el-button></el-col>
        <el-col :span="4" align="right">  <el-button type="primary" icon="el-icon-plus" circle @click="openEditUi(null)" /></el-col>
      </el-row>

    </el-card>

    <el-card>

      <el-table
        :data="userList"
        stripe
        style="width: 100%"
      >
        <el-table-column
          label="#"
          width="80"
        >
          <template slot-scope="scope">
            {{ (searchModel.pageNo-1) * searchModel.pageSize + scope.$index + 1 }}
          </template>
        </el-table-column>
        <el-table-column
          prop="username"
          label="用户名"
          width="180"
        />
        <el-table-column
          prop="userId"
          label="用户id"
          width="180"
        />
        <el-table-column
          prop="phone"
          label="电话"
          width="180"
        />
        <el-table-column
          label="用户状态"
          prop="status"
          width="180px"
        >
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status === 1">正常</el-tag>
            <el-tag v-else type="danger">禁用</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="email"
          label="邮箱"
        />
        <el-table-column
          label="操作"
          width="180px"
        >
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" circle size="mini" @click="openEditUi(scope.row.userId)" />
            <el-button type="danger" icon="el-icon-delete" circle size="mini" @click="deleteUser(scope.row)" />
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-pagination
      :current-page="searchModel.pageNo"
      :page-sizes="[5, 10, 20, 30]"
      :page-size="searchModel.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />

    <el-dialog :title="title" :visible.sync="dialogFormVisible" @close="clearForm">
      <el-form ref="userFormRef" :model="userForm" :rules="rules">

        <el-form-item label="用户名" :label-width="formLabelWidth" prop="username">
          <el-input v-model="userForm.username" autocomplete="off" />
        </el-form-item>

        <el-form-item label="密码" :label-width="formLabelWidth" prop="password">
          <el-input v-model="userForm.password" :disabled="disabled" type="password" autocomplete="off" />
        </el-form-item>

        <el-form-item label="电话" :label-width="formLabelWidth">
          <el-input v-model="userForm.phone" autocomplete="off" />
        </el-form-item>

        <el-form-item label="用户状态" :label-width="formLabelWidth">
          <el-switch
            v-model="userForm.status"
            :active-value="1"
            :inactive-value="0"
          />
        </el-form-item>

        <el-form-item label="邮箱地址" :label-width="formLabelWidth" prop="email">
          <el-input v-model="userForm.email" autocomplete="off" />
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveUser">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import userApi from '@/api/userManage'

export default {
  name: 'User',
  data() {
    return {

      formLabelWidth: '120px',
      userForm: {

      },
      disabled: false,
      dialogFormVisible: false,
      title: '',
      total: 0,
      searchModel: {
        username: '',
        phone: '',
        pageNo: 1,
        pageSize: 10
      },
      userList: [
        {
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        },
        {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1517 弄'
        },
        {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄'
        },
        {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1516 弄'
        }
      ],
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 20, message: '长度在 6 到 20 个字符', trigger: 'blur' }
        ],

        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' }

        ]

      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    deleteUser(user) {
      this.$confirm(`您确认删除用户 ${user.username} ?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        userApi.deleteUserById(user.userId).then(res => {
          this.$message({
            type: 'success',
            message: res.msg
          })

          this.getUserList()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    saveUser() {
      // 1. 表单校验

      this.$refs.userFormRef.validate((valid) => {
        if (valid) {
          // 2. 发送请求
          userApi.saveUser(this.userForm).then(res => {
            this.$message({
              type: 'success',
              message: res.msg
            })
            // 关闭弹出框
            this.dialogFormVisible = false
            // 刷新表格
            this.getUserList()
          })
        } else {
          console.log('error submit!!')
          return false
        }
      }

      )
    },

    clearForm() {
      this.userForm = {}
      this.$refs.userFormRef.clearValidate()
    },
    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize
      this.getUserList()
    },
    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo
      this.getUserList()
    },
    getUserList() {
      userApi.getUserList(this.searchModel).then(res => {
        this.userList = res.data.rows
        this.total = res.data.total
      })
    },
    openEditUi(id) {
      if (id == null) {
        this.title = '新增用户'
        this.disabled = false
        this.userForm.id = null
      } else {
        this.title = '编辑用户'
        userApi.getUserById(id).then(res => {
          this.userForm = res.data
          this.disabled = true
          this.userForm.id = id
        })
      }
      this.dialogFormVisible = true
    }
  }
}

</script>

<style lang="scss" scoped>
.box-card {
  width: 100%;
  margin-bottom: 20px;
  .el-input {
    width: 200px;
    margin-right: 20px;
  }
}
.el-dialog{
  .el-input{
    width: 85%;
  }
}
</style>
