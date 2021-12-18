<template>
  <div class="hello">
    <el-menu
      :default-active="$route.path"
      class="el-menu-demo"
      mode="horizontal"
      @select="handleSelect"
      background-color="#c55311"
      text-color="#fff"
      active-text-color="#ffd04b"
    >
      <el-menu-item index="1">
        <template slot="title">
          <i class="el-icon-refresh-left"></i>
        </template>
      </el-menu-item>
      <el-submenu index="2">
        <template slot="title">
          <i class="el-icon-more"></i>
          <span>添加</span>
        </template>
        <el-menu-item index="2-1">
          <template slot="title">
            <i class="el-icon-circle-plus-outline"></i>
            <span>添加队伍</span>
          </template>
        </el-menu-item>
        <el-menu-item index="2-2">
          <template slot="title">
            <i class="el-icon-circle-plus-outline"></i>
            <span>添加运动员</span>
          </template>
        </el-menu-item>
        <el-menu-item index="2-3">
          <template slot="title">
            <i class="el-icon-circle-plus-outline"></i>
            <span>添加教练</span>
          </template>
        </el-menu-item>
        <el-menu-item index="2-4">
          <template slot="title">
            <i class="el-icon-circle-plus-outline"></i>
            <span>添加队医</span>
          </template>
        </el-menu-item>
        <el-menu-item index="2-5">
          <template slot="title">
            <i class="el-icon-circle-plus-outline"></i>
            <span>添加领队</span>
          </template>
        </el-menu-item>
      </el-submenu>
      <el-menu-item index="3">
        <template slot="title">
          <i class="el-icon-circle-plus-outline"></i>
          <span>添加赛事</span>
        </template>
      </el-menu-item>
      <el-menu-item index="4">
        <template slot="title">
          <i class="el-icon-aim"></i>
          <span>运动员成绩</span>
        </template>
      </el-menu-item>
      <el-submenu index="5" class="admin">
        <template slot="title">
          <!-- <i class="el-icon-s-custom" ></i> -->
        </template>
        <el-menu-item index="5-1">
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
              <span>人员信息</span>
            </template>

            <!-- 所有队伍基本信息 -->
            <el-menu-item-group>
              <el-menu-item index="1-1">
                <i class="el-icon-user"></i>
                <span>所有队伍</span>
              </el-menu-item>
            </el-menu-item-group>

            <!-- 所有运动员基本信息 -->
            <el-menu-item-group>
              <el-menu-item index="1-2">
                <i class="iconfont icon-yundongyuan"></i>
                <span>所有运动员</span>
              </el-menu-item>
            </el-menu-item-group>

            <!-- 所有队医基本信息 -->
            <el-menu-item-group>
              <el-menu-item index="1-3">
                <i class="iconfont icon-yisheng"></i>
                <span>所有队医</span>
              </el-menu-item>
            </el-menu-item-group>

            <!--所有教练基本信息 -->
            <el-menu-item-group>
              <el-menu-item index="1-4">
                <i class="iconfont icon-jiaolian1"></i>
                <span>所有教练</span>
              </el-menu-item>
            </el-menu-item-group>
          </el-submenu>
          <!--初赛基本信息 -->
          <el-menu-item index="2">
            <template slot="title">
              <i class="el-icon-aim"></i>
              <span>初赛</span>
            </template>
          </el-menu-item>
          <!--决赛基本信息 -->
          <el-menu-item index="3">
            <template slot="title">
              <i class="el-icon-aim"></i>
              <span>决赛</span>
            </template>
          </el-menu-item>
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
      //队伍
      team: "",
      index: "",
    };
  },
  created() {
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
        //添加队伍
        case "2-1":
          this.$router.push("/addTeam");
          break;
        //添加运动员
        case "2-2":
          this.$router.push("/addAthlete");
          break;
        //添加教练
        case "2-3":
          this.$router.push("/addReferee");
          break;
        //添加队医
        case "2-4":
          this.$router.push("/addDoctor");
          break;
        //添加领队
        case "2-5":
          this.$router.push("/addLeader");
          break;
        //添加赛事
        case "3":
          this.$router.push("/addGame");
          break;
        //运动员成绩
        case "4":
          this.$router.push("/athleteScore");
          break;
        //退出登录  Athlete是前界面
        case "5-1":
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
        //所有队伍
        case "1-1":
          this.$router.push("/allTeam");
          break;
        //所有运动员
        case "1-2":
          this.$router.push("/allAthlete");
          break;
        //所有队医
        case "1-3":
          this.$router.push("/allDoctor");
          break;
        //所有教练员
        case "1-4":
          this.$router.push("/allReferee");
          break;
        //初赛
        case "2":
          this.$router.push("/preliminary");
          break;
        //决赛
        case "3":
          this.$router.push("/final");
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
.admin {
  display: block;
  position: absolute;
  right: 0;
  border-right: 0 !important;
}
</style>