<template>
  <el-container>
    <el-header>
      <div class="logo">
        <img src="../assets/logo.png" alt="logo" />
        <h1>电商后台管理系统</h1>
      </div>
      <el-button type="info" @click="exit">退出</el-button>
    </el-header>
    <el-container>
      <el-aside width="200px">
        <el-menu
          :default-active="paths"
          class="el-menu-vertical-demo"
          background-color="#464B55"
          text-color="#fff"
          active-text-color="#D7C083"
          unique-opened
          router
        >
          <el-submenu
            :index="item.id + ''"
            v-for="item in menuList"
            :key="item.id"
          >
            <template slot="title">
              <i :class="iconObj[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <el-menu-item-group>
              <el-menu-item
                :index="'/' + children.path"
                v-for="children in item.children"
                :key="children.id"
                @click="sessionSet('/' + children.path)"
              >
                <template slot="title">
                  <i class="el-icon-menu"></i>
                  <span>{{ children.authName }}</span>
                </template>
              </el-menu-item>
            </el-menu-item-group>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      menuList: [],
      iconObj: {
        125: 'iconfont icon-icon_group',
        103: 'iconfont icon-huiyuan',
        101: 'iconfont icon-gouwuche',
        102: 'iconfont icon-wodedingdan',
        145: 'iconfont icon-shuju'
      },
      paths: ''
    }
  },
  created() {
    this.getMenuList()
    this.paths = window.sessionStorage.getItem('sessionPath')
  },
  methods: {
    exit() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有的菜单
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
      console.log(this.menuList)
    },
    sessionSet(index) {
      window.sessionStorage.setItem('sessionPath', index)
      this.paths = window.sessionStorage.getItem('sessionPath')
    }
  }
}
</script>

<style lang="css" scoped>
.el-container {
  height: 100%;
}
.el-header {
  background-color: #464b55;
  height: 80px;
  display: flex;
  justify-content: space-between;
}
.el-aside {
  background-color: #464b55;
}
.el-main {
  background-color: #eceff7;
}
.logo {
  display: flex;
}
.logo img {
  width: 30px;
  height: 30px;
  border: 1px solid #ddd;
  box-shadow: 0 0 10px #ddd;
  border-radius: 50%;
  padding: 10px;
}
.logo h1 {
  color: #fff;
  font-size: 20px;
  display: flex;
  align-items: center;
  margin-left: 15px;
}
.el-header .el-button {
  height: 40px;
  margin-top: 10px;
}
.el-submenu {
  width: 200px;
}
</style>
