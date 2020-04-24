<template>
  <el-container class="home_container">
    <!-- 头部 -->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt />
        <span>商城后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="iscollapse?'64px':'200px'">
        <div class="toggle_button" @click="toggleMenu">|||</div>
        <el-menu
          class="el-menu-vertical-demo"
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409eff"
          unique-opened
          :collapse="iscollapse"
          :collapse-transition="false"
          :router = "true"
          :default-active="activePath"
        >
          <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconlist[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/'+subitem.path" v-for="subitem in item.children" :key="subitem.id" @click="savePath('/'+subitem.path)">
             <template slot="title">
              <i class="el-icon-menu"></i>
              <span>{{subitem.authName}}</span>
            </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 主体区域 -->
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
      menuList:[],
      iconlist:{
        '125':'iconfont icon-user',
        '103':'iconfont icon-tijikongjian',
        '101':'iconfont icon-shangpin',
        '102':'iconfont icon-danju',
        '145':'iconfont icon-baobiao'
      },
      iscollapse:false,
      activePath:''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      const {data:res} = await this.$http.get('menus')
      if (res.meta.status!==200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    toggleMenu () {
      this.iscollapse=!this.iscollapse
    },
    savePath (activePath) {
      window.sessionStorage.setItem('activePath',activePath)
      this.activePath = activePath
    }
  }
}
</script>
<style lang="less" scoped>
.home_container {
  height: 100%;
}
.el-header {
  background: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
    img{
      width: 60px;
    }
    > span {
      margin-left: 20px;
    }
  }
}
.el-container {
  overflow: hidden;
  background: #333744;
  .toggle_button{
    height: 30px;
    line-height: 30px;
    font-size: 10px;
    background: #4a5064;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
 .el-aside {
  background-color: #333744;
  .el-menu {
    border-right: none;
  }
}
}
.el-main {
  background: #eaedf1;
}
.iconfont{
  margin-right: 15px;
}
</style>