<template>
  <el-form
    ref="TeamFormRef"
    :model="TeamForm"
    label-width="35%"
    class="Form"
    :rules="TeamRules"
  >
    <div class="wrap">
      <h1>添加队伍信息</h1>
    </div>

    <el-form-item label="队伍Id" class="teamId" prop="teamId">
      <el-input
        v-model="TeamForm.teamId"
        placeholder="点击生成队伍Id"
        class="teamIdInput"
        :disabled="true"
      ></el-input>
      <el-button type="primary" @click="createId">生成</el-button>
    </el-form-item>

    <el-form-item label="队伍名称" class="name" prop="name">
      <el-input
        v-model="TeamForm.name"
        placeholder="请输入队伍名称"
        class="nameInput"
      ></el-input>
    </el-form-item>

    <el-form-item label="参赛日期" class="date" prop="date">
      <el-col :span="11">
        <el-date-picker
          type="date"
          placeholder="请选择报名时间"
          v-model="TeamForm.date"
          style="width: 100%"
          value-format="yyyy-MM-dd"
          format="yyyy 年 MM 月 dd 日"
        ></el-date-picker>
      </el-col>
    </el-form-item>

    <el-form-item label="队伍账号" class="account" prop="account">
      <el-input
        v-model="TeamForm.account"
        placeholder="请输入队伍账号"
        class="accountInput"
        @change="account"
      ></el-input>
    </el-form-item>

    <el-form-item label="队伍密码" class="password" prop="password">
      <el-input
        v-model="TeamForm.password"
        placeholder="请输入队伍密码"
        class="passwordInput"
        @change="password"
      ></el-input>
    </el-form-item>

    <el-form-item>
      <el-button type="primary" @click="onSubmit">立即添加</el-button>
      <el-button @click="exit">取消</el-button>
    </el-form-item>
  </el-form>
</template>
<script>
import routers from "../../router";

export default {
  data() {
    return {
      TeamForm: {
        teamId: "",
        name: "",
        account: "",
        password: "",
        date: "",
      },
      //验证规则
      TeamRules: {
        teamId: [{ required: true, message: "请生成队伍ID", trigger: "blur" }],
        name: [
          { required: true, message: "请输入队伍名称", trigger: "blur" },
          { min: 2, max: 7, message: "长度在2 ~ 7个字符", trihger: "blur" },
        ],
        account: [
          { required: true, message: "请输入队伍账号", trigger: "blur" },
          { min: 4, max: 8, message: "长度在4 ~ 8个字符", trihger: "blur" },
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 6, max: 18, message: "长度在6 ~ 18个字符", trihger: "blur" },
        ],
        date: [{ required: true, message: "请选择报名时间", trigger: "blur" }],
      },
    };
  },
  created() {
    this.keyupSubmit();
  },
  methods: {
    //键盘监视
    keyupSubmit() {
      document.onkeydown = (e) => {
        let _key = window.event.keyCode;
        if (_key === 13) {
          this.onSubmit();
        }
      };
    },
    account: function () {
      if (!/^[a-zA-Z0-9_-]{4,8}$/.test(this.TeamForm.account)) {
        this.$message({
          duration: 800,
          message: "请设置长度为4到8位可包含字母，数字，下划线，减号的账号",
          type: "error",
        });
        this.TeamForm.account = "";
        return;
      }
      //查询队伍账号是否存在
      this.$http
        .get("gymnastics/team-login/geta/" + this.TeamForm.account)
        .then((res) => {
          if (res.data) {
            this.$message({
              duration: 600,
              message: "队伍账号已存在，请重新设置账号",
              type: "error",
            });
            this.TeamForm.account = "";
            return;
          }
        });
    },
    password: function () {
      if (
        !/^(?=.*\d)(?=.*[a-zA-Z])[\da-zA-Z~!@#$%^&*]{6,18}$/.test(
          this.TeamForm.password
        )
      ) {
        this.$message({
          duration: 800,
          message: "请设置长度为6-18位且必须包含数字及英文的密码",
          type: "error",
        });
        this.TeamForm.password = "";
        return;
      }
    },
    createId() {
      //查找最后一个队伍编号
      this.$http.get("gymnastics/team/getlastteam").then((res) => {
        this.TeamForm.teamId = ++res.data[res.data.length - 1].teamId;
      });
    },
    onSubmit() {
      this.$refs.TeamFormRef.validate(async (valid) => {
        //验证失败
        if (!valid) return;
        console.log(this.TeamForm.teamId);
        var team = {
          teamId: this.TeamForm.teamId,
          teamName: this.TeamForm.name,
          registerYear: this.TeamForm.date,
          leaderId: null,
          doctorId: null,
          coachCount: 0,
        };
        //插入队伍表
        await this.$http.post("gymnastics/team", team).then((res) => {
          if (res.data.flag) {
            this.$message({
              duration: 600,
              message: "添加成功！",
              type: "success",
            });
          } else {
            this.$message({
              duration: 600,
              message: "添加失败",
              type: "error",
            });
            return;
          }
        });
        //新的队伍
        console.log(this.TeamForm.teamId);
        var teamLogin = {
          teamId: this.TeamForm.teamId,
          teamName: this.TeamForm.name,
          teamAccount: this.TeamForm.account,
          teamPassword: this.TeamForm.password,
          joinYear: this.TeamForm.date,
        };
        //插入新的队伍登录表
        await this.$http
          .post("gymnastics/team-login", teamLogin)
          .then((res) => {
            if (res.data) {
            } else {
              this.$message({
                duration: 600,
                message: "添加失败",
                type: "error",
              });
              return;
            }
          });

        console.log(this.TeamForm.teamId);
      });
    },
    navigate() {
      routers.go(-1);
    },
    exit() {
      this.$router.go(-1);
    },
  },
};
</script>
<style lang= "less" scoped>
.Form {
  display: block;
  position: absolute;
  align-items: center;
  top: 10%;
  bottom: 0;
  right: 0;
  width: 80%;
  height: 90%;
  background-color: rgb(193, 219, 208);
  overflow-x: scroll;
}
.wrap {
  height: 20%;
  width: 90%;
  display: table;
  text-align: center;
}
.teamId {
  height: 8%;
}
.teamIdInput {
  width: 30%;
}
.name {
  height: 8%;
}
.nameInput {
  width: 30%;
}
.account {
  height: 8%;
}
.accountInput {
  width: 30%;
}
.password {
  height: 8%;
}
.passwordInput {
  width: 30%;
}
.date {
  height: 8%;
}
</style>