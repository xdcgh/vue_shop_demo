<template>

  <el-container class="home-container">
    <el-header>
      <div>
        <img src="../assets/logo.png" alt="" width="64px">
        <span>麻雀生鲜后台管理系统</span>
      </div>

      <el-button type="info" @click="logout">退出</el-button>
    </el-header>

    <el-container>

      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>

        <el-menu
          background-color="#333744" text-color="#fff" active-text-color="#409eef"
          :unique-opened="true" :collapse="isCollapse" :collapse-transition="false"
          :router="true" :default-active="activePath">

          <!--    一级菜单      -->
          <el-submenu :index="item.id.toString()"
                      v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsObject[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>

            <!--    二级菜单        -->
            <el-menu-item :index="'/' + subItem.path" @click="saveNavState('/'+subItem.path)"
                          v-for="subItem in item.children" :key="subItem.id">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>

      <el-main>
        <!-- 路由占位符-->
        <router-view></router-view>
      </el-main>

    </el-container>
  </el-container>

</template>

<script>
  export default {
    data () {
      return {
        menuList: [],
        iconsObject: {
          '125': 'el-icon-user-solid',
          '103': 'el-icon-star-on',
          '101': 'el-icon-s-goods',
          '102': 'el-icon-s-help',
          '145': 'el-icon-s-tools'
        },
        // 是否折叠
        isCollapse: false,
        // 激活的菜单选项
        activePath: ''
      }
    },
    created () {
      this.getMenuList()

      this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods: {
      logout () {
        window.sessionStorage.clear()

        this.$router.push('/login')
      },
      async getMenuList () {
        const { data: res } = await this.$http.get('menus')

        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }

        this.menuList = res.data

        console.log(res)
      },
      // 菜单折叠与展开事件
      toggleCollapse () {
        this.isCollapse = !this.isCollapse
      },
      // 保存菜单选择的激活状态
      saveNavState (activePath) {
        window.sessionStorage.setItem('activePath', activePath)

        this.activePath = activePath
      }
    }
  }
</script>

<style lang="less" scoped>
  .home-container {
    height: 100%;
  }

  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: white;
    font-size: 20px;

    > div {
      display: flex;
      align-items: center;

      span {
        margin-left: 15px;
      }
    }
  }

  .el-aside {
    background-color: #333744;

    // 去掉二级菜单选项的外边框溢出
    .el-menu {
      border-right: none;
    }
  }

  .el-main {
    background-color: #eaedf1;
  }

  .toggle-button {
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: white;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }

</style>
