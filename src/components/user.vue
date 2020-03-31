<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row class="search" :gutter="20">
        <el-col :span="6">
          <el-input
            placeholder="请输入内容"
            class="input-select"
            v-model="queryInfo.query"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUsers"
            ></el-button>
          </el-input>
        </el-col>
        <el-button type="primary" @click="addUserHandle">添加用户</el-button>
      </el-row>
      <el-table :data="tableData" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <template v-slot:default="slotProps">
            <el-switch
              v-model="slotProps.row.mg_state"
              active-color="#13ce66"
              inactive-color="#ff4949"
              @change="switchChangeHandle(slotProps.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="slotId">
            <el-row type="flex" justiyf="center">
              <el-col>
                <!-- 删除按钮 -->
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="deleteUser(slotId.row.id)"
                ></el-button
              ></el-col>
              <el-col>
                <!-- 修改按钮 -->
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="editUsersHandle(slotId.row.id)"
                ></el-button
              ></el-col>
              <!-- 分配角色按钮 -->
              <el-col>
                <el-tooltip
                  class="item"
                  effect="dark"
                  content="设置角色"
                  placement="top-start"
                  :enterable="false"
                >
                  <el-button
                    type="warning"
                    icon="el-icon-s-tools"
                    size="mini"
                  ></el-button>
                </el-tooltip>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="totals"
      >
      </el-pagination>
    </el-card>
    <!-- 添加用户表单 -->
    <el-dialog
      title="添加用户"
      :visible.sync="dialogFormVisible"
      @close="closeAdduser"
    >
      <el-form :rules="addUserRules" :model="addUser" ref="addUserRef">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addUser.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addUser.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addUser.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addUser.mobile"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addUserCancle">取 消</el-button>
        <el-button type="primary" @click="addUserSure">确 定</el-button>
      </div>
    </el-dialog>
    <el-dialog title="编辑用户" :visible.sync="dialogEditVisible" width="40%">
      <el-form :model="editUser" :rules="editUsersRules" ref="editUsersRefs">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editUser.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editUser.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editUser.mobile"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogEditVisible = false">取 消</el-button>
        <el-button type="primary" @click="sureEidtHanle">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      // 邮箱验证规则
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) return callback()
      callback(new Error('请输入正确的邮箱'))
    }
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) return callback()
      callback(new Error('请输入正确的手机号'))
    }
    return {
      tableData: [],
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      totals: 0,
      dialogFormVisible: false,
      dialogEditVisible: false,
      addUser: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editUser: {},
      addUserRules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 10,
            message: '用户名长度在3-10个字符',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 6,
            max: 15,
            message: '密码长度在6-15个字符',
            trigger: 'blur'
          }
        ],
        email: [
          {
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          },
          {
            validator: checkEmail,
            trigger: 'blur'
          }
        ],
        mobile: [
          {
            required: true,
            message: '请输入手机',
            trigger: 'blur'
          },
          {
            validator: checkMobile,
            trigger: 'blur'
          }
        ]
      },
      editUsersRules: {
        email: [
          {
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          },
          {
            validator: checkEmail,
            trigger: 'blur'
          }
        ],
        mobile: [
          {
            required: true,
            message: '请输入手机',
            trigger: 'blur'
          },
          {
            validator: checkMobile,
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getUsers()
  },
  methods: {
    async getUsers() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('用户列表失败')
      this.tableData = res.data.users
      this.totals = res.data.total
    },
    // 改变当前显示条数
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUsers()
    },
    // 改变当前显示页数
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getUsers()
    },
    // 更改用户状态
    async switchChangeHandle(val) {
      console.log(val)
      const { data: res } = await this.$http.put(
        `users/${val.id}/state/${val.mg_state}`
      )
      console.log(res)
      if (res.meta.status !== 200) {
        val.mg_state = !val.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户成功')
    },
    // 点击添加用户事件
    addUserHandle() {
      this.dialogFormVisible = true
    },
    // 点击取消添加用户
    addUserCancle() {
      this.$refs.addUserRef.resetFields()
      this.dialogFormVisible = false
    },
    // 点击确定添加用户
    addUserSure() {
      this.$refs.addUserRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addUser)
        console.log(res)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败！')
        }

        this.$message.success('添加用户成功！')
        // 隐藏添加用户的对话框
        this.dialogFormVisible = false
        // 重新获取用户列表数据
        this.getUsers()
      })
    },
    closeAdduser() {
      this.$refs.addUserRef.resetFields()
    },
    // 删除用户
    deleteUser(val) {
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async suc => {
          // 确定删除的逻辑
          const { data: res } = await this.$http.delete(`users/${val}`)
          if (res.meta.status !== 200) {
            return this.$message.error('删除用户失败')
          }
          this.$message.success('删除成功')
          this.getUsers()
        })
        .catch(() => {
          // 取消的逻辑
          this.$message.info('已取消删除')
        })
    },
    // 点击修改用户按钮
    async editUsersHandle(val) {
      this.dialogEditVisible = true
      const { data: res } = await this.$http.get(`users/${val}`)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }
      this.editUser = res.data
    },
    // 确认修改用户
    sureEidtHanle() {
      this.$refs.editUsersRefs.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'users/' + this.editUser.id,
          {
            email: this.editUser.email,
            mobile: this.editUser.mobile
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更新失败')
        }
        this.$message.success('更新成功')
        this.dialogEditVisible = false
        this.getUsers()
      })
    }
  }
}
</script>
<style lang="css" scoped>
.el-breadcrumb {
  line-height: 1.5;
  margin-bottom: 10px;
}
.search {
  margin-bottom: 15px;
}
.el-pagination {
  margin-top: 15px;
}
</style>
