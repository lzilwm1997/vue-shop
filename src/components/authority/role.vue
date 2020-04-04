<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片容器 -->
    <el-card>
      <!-- 添加用户 -->
      <el-button type="primary" @click="addRole" class="addButStyle">
        添加角色
      </el-button>
      <!-- 表格 -->
      <el-table border style="width: 100%;" :data="rolesList">
        <!-- 角色详情权限扩展 -->
        <el-table-column type="expand">
          <template v-slot:default="roleExpand">
            <!-- <pre>{{ roleExpand.row }}</pre> -->
            <!-- 第一级权限 -->
            <el-row v-for="(item1, index1) in roleExpand.row.children" :key="item1.id" :class="[
                'borderBot',
                'colCenter',
                index1 === 0 ? 'borderTop' : '',
              ]">
              <el-col :span="4">
                <el-tag type="" closable @close="deleteRoleTag(roleExpand.row, item1.id)">
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二三级权限 -->
              <el-col :span="20">
                <el-row v-for="(item2, index2) in item1.children" :key="item2.id" :class="[
                    '.borderTop',
                    'colCenter',
                    index2 === 0 ? '' : 'borderTop',
                  ]">
                  <!-- 二级权限 -->
                  <el-col :span="4">
                    <el-tag type="success" closable class="tagPad" @close="deleteRoleTag(roleExpand.row, item2.id)">
                      {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="20" class="tagPad">
                    <el-tag type="warning" closable v-for="item3 in item2.children" :key="item3.id" class="rightPad"
                      @close="deleteRoleTag(roleExpand.row, item3.id)">{{ item3.authName }}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="slotRoleId">
            <el-button type="primary" size="mini" icon="el-icon-edit" @click="editRole(slotRoleId.row)">编辑</el-button>
            <el-button type="danger" size="mini" icon="el-icon-delete" @click="deleteRole(slotRoleId.row)">删除
            </el-button>
            <el-button type="warning" size="mini" icon="el-icon-s-tools" @click="allotHandle(slotRoleId.row)">分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色表单 -->
    <el-dialog title="添加角色" :visible.sync="addRoleVisible" width="30%">
      <el-form :model="addRoleData" :rules="Addrules" ref="addRoleRefs">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleData.roleName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleData.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addRoleCancle">取 消</el-button>
        <el-button type="primary" @click="addRoleSure">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 编辑角色表单 -->
    <el-dialog title="编辑角色" :visible.sync="editRoleVisible" width="30%">
      <el-form :model="editRoleData" :rules="Addrules" ref="editRoleRefs">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleData.roleName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleData.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editRoleCancle">取 消</el-button>
        <el-button type="primary" @click="editRoleSure">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 分配权限窗口 -->
    <el-dialog title="分配角色" :visible.sync="allotDialogVisible" width="30%" @close="allotDialogClose">
      <span>
        <!-- 分配权限树状图 -->
        <el-tree :data="rolesTreeData" show-checkbox node-key="id" :props="rolesTreeProps" default-expand-all
          :default-checked-keys="defCheck" ref="rolesTreeRefs">
        </el-tree>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="allotDialogCancel">取 消</el-button>
        <el-button type="primary" @click="allotDialogSure">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        // 角色列表数据
        rolesList: [],
        // 树状图角色权限
        rolesTreeData: [],
        // 角色默认选中权限
        defCheck: [],
        // 当前点击的角色id值
        roleID: '',
        rolesTreeProps: {
          children: 'children',
          label: 'authName'
        },
        addRoleData: {
          roleName: '',
          roleDesc: ''
        },
        // 默认隐藏各dialog弹窗组件
        addRoleVisible: false,
        allotDialogVisible: false,
        editRoleVisible: false,
        editRoleData: {},
        titleRole: '',
        // 添加角色的验证规则
        Addrules: {
          roleName: [{
              required: true,
              message: '请输入角色名称',
              trigger: 'blur'
            },
            {
              min: 2,
              max: 15,
              message: '长度在 2 到 15 个字符',
              trigger: 'blur'
            }
          ],
          roleDesc: [{
            min: 2,
            max: 20,
            message: '长度在 2 到 20 个字符',
            trigger: 'blur'
          }]
        }
      }
    },
    created() {
      this.getRolesList()
    },
    methods: {
      async getRolesList() {
        const {
          data: res
        } = await this.$http.get('roles')
        //   console.log(res)
        if (res.meta.status !== 200) {
          return this.message.error('角色列表获取失败')
        }
        this.rolesList = res.data
      },
      // 编辑角色
      async editRole(val) {
        this.editRoleVisible = true
        const {
          data: res
        } = await this.$http.get(`roles/${val.id}`)
        if (res.meta.status !== 200) {
          this.$message.error('获取角色信息失败')
        }
        this.editRoleData = res.data
      },
      editRoleCancle() {
        this.$refs.editRoleRefs.resetFields()
        this.editRoleVisible = false
      },
      editRoleSure() {
        console.log(this.editRoleData)
        this.$refs.editRoleRefs.validate(async (valid) => {
          if (!valid) return
          const {
            data: res
          } = await this.$http.put(
            `roles/${this.editRoleData.roleId}`,
            this.editRoleData
          )
          if (res.meta.status !== 200) {
            return this.$message.error('编辑角色失败')
          }
          this.$refs.editRoleRefs.resetFields()
          this.editRoleVisible = false
          this.getRolesList()
          this.$message.success('编辑角色成功')
        })
      },
      // 删除角色
      deleteRole(val) {
        // console.log(val)
        this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          })
          .then(async (suc) => {
            const {
              data: res
            } = await this.$http.delete(`roles/${val.id}`)
            if (res.meta.status !== 200) {
              return this.$message.error('删除用户失败')
            }
            this.$message.success('删除成功')
            this.getRolesList()
          })
          .catch(() => {
            this.$message.info('已取消删除')
          })
      },
      // 添加角色
      addRole() {
        this.addRoleVisible = true
      },
      addRoleCancle() {
        this.addRoleVisible = false
        this.$refs.addRoleRefs.resetFields()
      },
      addRoleSure() {
        this.$refs.addRoleRefs.validate(async (valid) => {
          if (!valid) return
          const {
            data: res
          } = await this.$http.post('roles', this.addRoleData)
          console.log(res)
          if (res.meta.status !== 201) {
            return this.$message.error('添加角色失败')
          }
          this.$refs.addRoleRefs.resetFields()
          this.addRoleVisible = false
          this.getRolesList()
          this.$message.success('添加角色成功')
        })
      },
      async deleteRoleTag(role, right) {
        const confirmResult = await this.$confirm(
          '此操作将永久删除该权限, 是否继续?',
          '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }
        )
        if (confirmResult !== 'confirm') {
          return this.$message.error('取消删除成功')
        }
        const {
          data: res
        } = await this.$http.delete(
          `roles/${role.id}/rights/${right}`
        )
        if (res.meta.status !== 200) {
          return this.$message.error('删除权限失败')
        }
        this.$message.success('删除成功')
        role.children = res.data
      },
      // 分配权限
      async allotHandle(val) {
        this.roleID = val.id
        const {
          data: res
        } = await this.$http.get('rights/tree')
        if (res.meta.status !== 200) {
          return this.$message.error('获取权限数据失败！')
        }
        console.log(res)
        console.log(val)
        this.rolesTreeData = res.data
        this.getDefCheckList(val, this.defCheck)
        this.allotDialogVisible = true
      },
      // 获取默认勾选角色权限的数组
      getDefCheckList(node, arr) {
        if (!node.children) {
          return arr.push(node.id)
        }
        node.children.forEach((element) => {
          this.getDefCheckList(element, arr)
        })
      },
      // 关闭弹窗时重置数组
      allotDialogClose() {
        this.defCheck = []
      },
      // 点击确定更新勾选权限
      async allotDialogSure() {
        const getCheckedKeys = this.$refs.rolesTreeRefs.getCheckedKeys()
        const keys = this.$refs.rolesTreeRefs.getHalfCheckedKeys().concat(getCheckedKeys)
        const keysStr = keys.toString()
        const {
          data: res
        } = await this.$http.post(`roles/${this.roleID}/rights`, {
          rids: keysStr
        })
        if (res.meta.status !== 200) {
          return this.$message.success('分配权限失败')
        }
        this.$message.success('分配权限成功')
        this.getRolesList()
        this.allotDialogVisible = false
      },
      // 点击取消分配权限
      allotDialogCancel() {
        this.$message.info('取消分配权限成功')
        this.allotDialogVisible = false
      }
    }
  }

</script>
<style lang="css" scoped>
  .borderBot {
    border-bottom: 1px solid #ebeef5;
  }

  .colCenter {
    display: flex;
    align-items: center;
  }

  .borderTop {
    border-top: 1px solid #ebeef5;
  }

  .tagPad {
    margin: 10px 0;
  }

  .rightPad {
    margin-right: 5px;
    margin-bottom: 5px;
  }

  .addButStyle {
    margin-bottom: 15px;
  }

</style>
