<template>
  <div class="layout">
    <div class="siderbar-wrapper" :style="{width: this.isCollapsed?'64px':'200px'}">
      <div class="logo-wrapper">logo</div>
      <el-menu
        background-color="#424f63"
        text-color="#fff"
        active-text-color="#65cea7"
        class="menu-wrapper"
        router
        unique-opened
        :collapse="isCollapsed"
        :default-active="$route.path"
      >
        <template v-for="(item, index) in sider_menu_data">
          <el-menu-item class="menu-item" v-if="!item.children" :index="item.path" :key="index">
            <i :class="item.icon"></i>
            <span slot="title">{{item.title}}</span>
          </el-menu-item>
          <el-submenu v-else :index="item.path">
            <template slot="title">
              <i :class="item.icon"></i>
              <span slot="title">{{item.title}}</span>
            </template>
            <el-menu-item
              class="menu-item"
              v-for="(sub_item, sub_index) in item.children"
              :index="sub_item.path"
              :key="sub_index"
            >
              <span slot="title" style="margin-left:12px;">{{sub_item.title}}</span>
            </el-menu-item>
          </el-submenu>
        </template>
      </el-menu>
    </div>
    <div class="topbar-wrapper" :style="{left: this.isCollapsed?'64px':'200px'}">
      <div class="menu-collapse-wrapper float-left" @click="toggleMenu">
        <i
          class="el-icon-adm-menu"
          :style="{transform: 'rotateZ(' + (this.isCollapsed ? '90' : '0') + 'deg)'}"
        ></i>
      </div>
      <div class="title float-left">下田乐乐后台管理系统</div>
      <ul class="menu-list float-right">
        <li v-if="user" class="menu-item" style="padding: 0;">
          <el-dropdown
            :show-timeout="10"
            :hide-timeout="10"
            @command="handleCommand"
            style="padding: 0 15px;"
          >
            <div class="dropdown-content el-dropdown-link">
              <i class="icon el-icon-adm-usersetup"></i>
              <span class="text">{{user.name || user.username}}</span>
            </div>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="a">{{user.username}}</el-dropdown-item>
              <el-dropdown-item command="b">{{user.name}}</el-dropdown-item>
              <el-dropdown-item command="b">{{user.type.name}}</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </li>
        <!-- <li class="menu-item">
          <i class="icon el-icon-adm-help"></i>
        </li>-->
        <li class="menu-item" @click="exit">
          <i class="icon iconfont el-icon-adm-exit"></i>
        </li>
      </ul>
    </div>
    <div
      class="content-wrapper"
      ref="content-wrapper"
      :style="{left: this.isCollapsed?'64px':'200px'}"
    >
      <div class="content">
        <keep-alive>
          <router-view v-if="$route.meta.keepAlive"></router-view>
        </keep-alive>
        <router-view v-if="!$route.meta.keepAlive"></router-view>
      </div>
    </div>
  </div>
</template>
<script>
import { sessionStorage } from "src/assets/js/storage";

export default {
  created() {
    // this.checkAuth();
  },
  data() {
    return {
      sider_menu_data: [
        {
          path: "/home",
          title: "管理员",
          icon: "el-icon-adm-home",
          children: [
            { path: "/home/identity", title: "账号管理" },
            { path: "/home/power", title: "权限管理" },
            { path: "/home/goods", title: "商品信息" },
            { path: "/home/set", title: "商品设置"}
          ]
        },
        {
          path: "/tables",
          title: "表格管理",
          icon: "el-icon-adm-linechart",
          children: [
            { path: "/tables/basic", title: "基本表格" },
            { path: "/tables/sort", title: "排序表格" },
            { path: "/tables/filter", title: "筛选表格" }
          ]
        },
        {
          path: "/charts",
          title: "图表管理",
          icon: "el-icon-adm-statistics",
          children: [
            { path: "/charts/bar", title: "柱状图" },
            { path: "/charts/line", title: "折线图" },
            { path: "/charts/pie", title: "饼图" }
          ]
        },
        {
          path: "/form",
          title: "表单管理",
          icon: "el-icon-adm-form",
          children: [{ path: "/form/render", title: "渲染表单" }]
        },
        {
          path: "/system/index",
          title: "系统管理",
          icon: "el-icon-adm-project"
        },
        {
          path: "/user/index",
          title: "用户管理",
          icon: "el-icon-adm-user"
        },
        // {
        //   path: '/access/index',
        //   title: '权限管理',
        //   icon: 'el-icon-adm-organization'
        // },
        {
          path: "/log/index",
          title: "操作日志",
          icon: "el-icon-adm-log"
        },
        {
          path: "/test/index",
          title: "测试",
          icon: "el-icon-adm-formsetup"
        }
      ],
      isCollapsed: false,
      adminMenuShow: false
    };
  },
  computed: {
    user() {
      return this.$store.state.user;
    }
  },
  methods: {
    handleCommand(command) {
      console.log(command);
    },
    toggleMenu() {
      this.isCollapsed = !this.isCollapsed;
    },
    exit() {
      this.$confirm("即将退出系统登陆，是否继续？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.axios.post("/logout").then(res => {
            if (res.code == 1) {
              this.$router.replace({ path: "/login" });
            }
          }).catch();
          this.$store.commit('SET_TOKEN', '');
          this.$store.commit('SET_USER', null);
        })
        .catch(() => {
          return false;
        });
    }
  }
};
</script>
<style lang="scss">
@import "../../assets/styles/variable";
.siderbar-wrapper {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  width: $siderbarWidth;
  z-index: 11;
  background-color: $siderbarBackgroundColor;
  transition: all 0.3s ease-in-out;
  .logo-wrapper {
    height: 40px;
    line-height: 40px;
    padding: 16px 0;
    text-align: center;
    font-size: 24px;
    color: #ffffff;
    color: #65cea7;
  }
  .menu-wrapper {
    position: absolute;
    top: 72px;
    bottom: 0;
    width: 100%;
    border-right: none;
    transition: all 0.3s ease-in-out;
    &:not(.el-menu--collapse) {
      overflow-y: auto;
      overflow-x: hidden;
    }
    i {
      color: #ffffff;
    }
    .menu-item {
      &.is-active,
      &:hover {
        background-color: #353f4f !important;
        color: #65cea7 !important;
        i {
          color: #65cea7 !important;
        }
      }
    }
    .el-submenu__title:hover {
      color: #65cea7 !important;
      i {
        color: #65cea7 !important;
      }
    }
    .el-submenu,
    .el-menu-item {
      width: 100%;
    }
  }
}
.topbar-wrapper {
  position: fixed;
  left: $siderbarWidth;
  right: 0;
  top: 0;
  height: $topbarHeight;
  line-height: $topbarHeight;
  padding: 0 24px 0 0;
  background-color: $topbarBackgroundColor;
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.12), 0 0 6px 0 rgba(0, 0, 0, 0.04);
  transition: all 0.3s ease-in-out;
  z-index: 12;
  .menu-collapse-wrapper {
    height: 100%;
    width: 48px;
    text-align: center;
    cursor: pointer;
    i {
      transition: all 0.3s ease-in-out;
    }
  }
  .title {
    height: 100%;
    font-weight: bold;
  }
  .menu-list {
    .menu-item {
      position: relative;
      float: left;
      padding: 0 15px;
      min-width: 45px;
      height: 48px;
      text-align: center;
      font-size: 0px;
      &:hover {
        cursor: pointer;
        background-color: #f5f5f5;
      }
      .icon {
        vertical-align: middle;
        font-size: 24px;
      }
      .text {
        display: inline-block;
        vertical-align: middle;
        margin-left: 4px;
        font-size: 16px;
      }
    }
  }
}
.content-wrapper {
  position: fixed;
  left: $siderbarWidth;
  right: 0;
  top: $topbarHeight;
  bottom: 0;
  padding: 16px;
  overflow: auto;
  transition: all 0.3s ease-in-out;
  .content {
    position: relative;
    width: 100%;
    height: 100%;
  }
}
</style>
