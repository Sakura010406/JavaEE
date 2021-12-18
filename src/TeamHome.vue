<template>
  <div class="hello">
    <el-menu
      :default-active="$route.path"
      class="el-menu-demo"
      mode="horizontal"
      @select="handleSelect"
      background-color="#085563"
      text-color="#fff"
      active-text-color="#ffd04b"
    >
      <el-menu-item index="1">
        <template slot="title">
          <i class="el-icon-refresh-left"></i>
        </template>
      </el-menu-item>

      <el-submenu index="2" class="admin">
        <template slot="title">
          <!-- <i class="el-icon-s-custom" ></i> -->
        </template>
        <el-menu-item index="2-1">
          <template slot="title">
            <i class="el-icon-notebook-2"></i>
            <span>队伍信息</span>
          </template>
        </el-menu-item>
        <el-menu-item index="2-2">
          <template slot="title">
            <i class="el-icon-loading"></i>
            <span>退出登录</span>
          </template>
        </el-menu-item>
      </el-submenu>
    </el-menu>
    <!-- 侧拉列表 -->
    <el-row class="tac">
      <el-col :span="null">
        <el-menu
          :default-active="$route.path"
          :unique-opened="false"
          class="el-menu-vertical-demo"
          @open="handleOpen"
          @close="handleClose"
          @select="handleSelect1"
          :default-openeds="selectedIndexs"
          background-color="#fefeff"
          text-color="#010002"
          active-text-color="#8f0a3d"
          inactive-text-coloe="#010002"
        >
          <el-submenu index="1">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>我的队伍</span>
            </template>

            <!-- 领队信息 一张表单 -->
            <el-menu-item-group>
              <el-menu-item index="1-1">
                <i class="el-icon-user"></i>
                <span>领队信息</span>
              </el-menu-item>
            </el-menu-item-group>

            <!-- 队医信息 一张表单 -->
            <el-menu-item-group>
              <el-menu-item index="1-2">
                <i class="iconfont icon-yisheng"></i>
                <span>队医信息</span>
              </el-menu-item>
            </el-menu-item-group>

            <!-- 运动员信息 列表-->
            <el-menu-item-group>
              <el-menu-item index="1-3">
                <i class="iconfont icon-yundongyuan"></i>
                <span>运动员表</span>
              </el-menu-item>
            </el-menu-item-group>

            <!--教练信息 列表-->
            <el-menu-item-group>
              <el-menu-item index="1-4">
                <i class="iconfont icon-jiaolian1"></i>
                <span>教练员表</span>
              </el-menu-item>
            </el-menu-item-group>
          </el-submenu>
        </el-menu>
      </el-col>
    </el-row>

    <el-main style="background-color: rgb(193, 219, 208)">
      <router-view></router-view>
    </el-main>
  </div>
</template>
<script>
import routers from "../../router";
export default {
  inject: ["reload"],

  data() {
    return {
      selectedIndexs: ["1"],
      //activeIndex: "1",
      //队伍
      teamId: "",
      index: "",
      id: "",
      portrait: "",
      //头像
      adatar: "",
      //队伍账号
      teamAccount: "",
    };
  },
  created() {
    this.teamAccount = this.$route.query.teamAccount;

    //根据账号查询队伍Id
    this.$http
      .get("gymnastics/team-login/geta/" + this.teamAccount)
      .then((res) => {
        this.teamId = res.data.teamId;
      });
    this.$nextTick(() => {
      // 禁用右键
      document.oncontextmenu = new Function("event.returnValue=false");
      // 禁用选择
      document.onselectstart = new Function("event.returnValue=false");
    });
  },
  methods: {
    navigate() {
      routers.go(-1);
    },
    //上方导航栏
    handleSelect(key) {
      switch (key) {
        case "1":
          this.reload();
          this.$message({
            duration: 800,
            message: "刷新成功！",
            type: "success",
          });
          break;
        //队伍信息
        case "2-1":
          this.$router.push({
            path: "/team",
            query: { teamId: this.teamId },
          });
          break;
        //退出登录  
        case "2-2":
          this.$message({
            duration: 600,
            message: "已退出,请登录！",
            type: "success",
          });
          window.sessionStorage.clear();
          this.$router.push({ path: "/login" });
          break;
        default:
          break;
      }
    },
    //左侧导航栏
    handleSelect1(key) {
      switch (key) {
        //领队信息
        case "1-1":
          this.$router.push({
            path: "/teamLeader",
            query: { teamId: this.teamId },
          });
          break;
        //队医信息
        case "1-2":
          this.$router.push({
            path: "/teamDoctor",
            query: { teamId: this.teamId },
          });
          break;
        //运动员表
        case "1-3":
          this.$router.push({
            path: "/teamAthlete",
            query: { teamId: this.teamId },
          });
          break;
        //教练员表
        case "1-4":
          this.$router.push({
            path: "/teamReferee",
            query: { teamId: this.teamId },
          });
          break;

        default:
          break;
      }
    },
    handleOpen(key) {},
    handleClose(key) {},
  },
};
</script>
<style lang="less" scoped>
.body {
  background-color: rgb(193, 219, 208);
}
.el-menu-demo {
  display: block;
  position: absolute;
  top: 0;
  width: 100%;
  height: 10%;
  background-color: #109b66;
}
.tac {
  display: block;
  position: absolute;
  left: 0;
  height: 90%;
  bottom: 0;
  background-color: #fefeff;
  border-right: 0 !important;
  width: 20%;
}
.admin {
  display: block;
  position: absolute;
  right: 0;
  border-right: 0 !important;
}
</style>