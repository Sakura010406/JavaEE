<template>
  <div class="hello">
    <el-menu
      :default-active="$route.path"
      class="el-menu-demo"
      mode="horizontal"
      @select="handleSelect"
      background-color="#500808"
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
          <el-menu-item index="1">
            <i class="el-icon-edit"></i>
            <span>打分</span>
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
      activeIndexs: [""],
      index: "",
      //activeIndex: "1",
      //裁判
      teamAccount: "",
      refereeId: "",
    };
  },
  created() {
    //教练账号
    this.teamAccount = this.$route.query.teamAccount;

    //根据账号查询裁判Id
    this.$http
      .get("gymnastics/referee/getbyRefereeloginaccount/" + this.teamAccount)
      .then((res) => {
        this.refereeId = res.data.refereeId;
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

        //退出登录
        case "2-1":
          this.$message({
            duration: 600,
            message: "已退出,请登录！",
            type: "success",
          });
          //消除令牌
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
        //打分
        case "1":
          this.$router.push({
            path: "/score",
            query: { refereeId: this.refereeId },
          });
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