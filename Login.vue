<template>
  <div class="login_container">
    <img src="../image/2.jpg" class="bodyImg" />
    <!-- 登录块 -->
    <div class="login_box">
      <div class="avater_box">
        <img src="../image/9.gif" />
      </div>
      <!-- 表单区域 -->
      <el-form
        ref="loginFormRef"
        :rules="loginRules"
        :model="loginForm"
        class="login_form"
        label-width="0"
      >
        <h2 class="title">Max</h2>

        <!-- 队伍Account -->
        <el-form-item>
          <el-input
            v-model="loginForm.account"
            prefix-icon="iconfont icon-denglu"
            placeholder="队伍账号"
            @change="account"
            style="color: blue"
          ></el-input>
        </el-form-item>
        <!-- 队伍密码 -->
        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            prefix-icon="iconfont icon-mima"
            type="password"
            show-password
            placeholder="队伍密码"
            @change="password"
          ></el-input>
        </el-form-item>
        <!--选择身份-->
        <el-form-item prop="identify" ckass="identify">
          <el-row>
            <el-col :span="8">
              <el-input
                v-model="loginForm.identify"
                prefix-icon="iconfont icon-yuangong"
                placeholder="您的身份"
                :disabled="true"
              >
              </el-input>
            </el-col>
            <el-col :span="3">
              <el-dropdown
                type="primary"
                @command="handleCommand"
                trigger="click"
              >
                <el-button type="primary" class="identity">
                  <i class="el-icon-arrow-down el-icon--right"></i>
                </el-button>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="赛事方" divided
                    >赛事方</el-dropdown-item
                  >
                  <el-dropdown-item command="队伍方" divided
                    >队伍方</el-dropdown-item
                  >
                  <el-dropdown-item command="教练方" divided
                    >教练方</el-dropdown-item
                  >
                </el-dropdown-menu>
              </el-dropdown>
            </el-col>
          </el-row>
        </el-form-item>

        <!--不想链接 <router-link to="Register.vue">注册</router-link> -->
        <!-- 按钮 -->
        <el-form-item class="btns">
          <el-button
            type="primary"
            @click="login"
            icon="iconfont icon-tijiao"
            round
            >登录</el-button
          >
          <el-button
            type="primary"
            @click="resetLoginForm"
            icon="iconfont icon-zhongzhi"
            round
            >重置</el-button
          >
          <el-button
            type="primary"
            @click="register"
            icon="iconfont icon-zhuce"
            round
            >注册</el-button
          >
          <el-button
            type="primary"
            @click="findPassword"
            icon="iconfont icon-wangjimima"
            round
            >忘记密码？</el-button
          >
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      //表单数据
      loginForm: {
        account: "",
        password: "",
        identify: "",
      },
      id: "",
      team: "",
      //校验规则
      loginRules: {
        //blur 验证时机
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 1, max: 18, message: "长度在1 ~ 18个字符", trihger: "blur" },
        ],
      },
    };
  },
  created() {
    this.keyupSubmit();
  },
  methods: {
    //重置表单内容
    resetLoginForm() {
      this.loginForm.account = "";
      this.loginForm.password = "";
      // this.$refs.loginFormRef.resetFields();
    },
    //键盘监视
    keyupSubmit() {
      document.onkeydown = (e) => {
        let _key = window.event.keyCode;
        if (_key === 13) {
          this.login();
        }
      };
    },
    account: function () {
      if (!/^[a-zA-Z0-9_-]{4,8}$/.test(this.loginForm.account)) {
        this.$message({
          duration: 800,
          message: "请输入正确的账号",
          type: "error",
        });
        this.loginForm.account = "";
        return;
      }
    },
    password: function () {},
    //验证登录信息
    login() {
      //账号是否为空
      if (this.loginForm.account == "") {
        this.$message({
          duration: 800,
          message: "账号不能为空！",
          type: "error",
        });
        return;
      }
      //密码是否为空
      if (this.loginForm.password == "") {
        this.$message({
          duration: 800,
          message: "密码不能为空！",
          type: "error",
        });
        return;
      }
      //身份是否为空
      if (this.loginForm.identify == "") {
        this.$message({
          duration: 800,
          message: "身份不能为空！",
          type: "error",
        });
        return;
      }

      //验证校验规则
      this.$refs.loginFormRef.validate(async (valid) => {
        //验证失败
        if (!valid) return;

        if (this.loginForm.identify == "赛事方") {
          await this.$http
            .get(
              "gymnastics/manager-login/" +
                +this.loginForm.account.toString() +
                "/" +
                this.loginForm.password.toString()
            )
            .then((res) => {
              if (res.data) {
                //信息提示
                this.$message({
                  duration: 800,
                  message: "登录成功！",
                  type: "success",
                });
                sessionStorage.setItem("token", "true");
                this.$router.push("/manager");
              } else {
                this.$message({
                  duration: 800,
                  message: "账号或密码错误！",
                  type: "error",
                });
              }
            });
        } else if (this.loginForm.identify == "队伍方") {
          const { data: res } = await this.$http.get(
            "gymnastics/team-login/" +
              this.loginForm.account.toString() +
              "/" +
              this.loginForm.password.toString()
          );
          if (res) {
            //信息提示
            this.$message({
              duration: 800,
              message: "登录成功！",
              type: "success",
            });
            //页面跳转：带参数
            sessionStorage.setItem("token", "true");

            this.$router.push({
              path: "/team",
              query: { teamAccount: this.loginForm.account },
            });
          } else {
            this.$message({
              duration: 800,
              message: "账号或密码错误！",
              type: "error",
            });
          }
          return;
        } else {
          const { data: res } = await this.$http.get(
            "gymnastics/referee/getbyRefereeloginaccount/" +
              this.loginForm.account.toString()
          );
          if (res.refereeName == this.loginForm.password.toString()) {
            //信息提示
            this.$message({
              duration: 800,
              message: "登录成功！",
              type: "success",
            });
            //页面跳转：带参数
            sessionStorage.setItem("token", "true");
            this.$router.push({
              path: "/referee",
              query: { teamAccount: this.loginForm.account },
            });
          } else {
            this.$message({
              duration: 800,
              message: "账号或姓名错误！",
              type: "error",
            });
            return;
          }
        }
      });
    },
    handleCommand(command) {
      this.loginForm.identify = command;
    },
    //注册
    register() {
      this.$router.push("/register");
    },
    findPassword() {
      this.$router.push("/findpassword");
    },
  },
};
</script>

<style lang="less" scoped>
.login_container {
  background-color: rgb(193, 219, 208);
  height: 100%;
}
.login_box {
  opacity: 0.8;
  width: 480px;
  height: 300px;
  background-color: #fff;
  border-radius: 10px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  .avater_box {
    width: 130px;
    height: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 5px;
    box-shadow: 0 0 5px #ddd;
    position: absolute;
    left: 50%;
    bottom: 80%;
    transform: translate(-50%, -50%);
    background-color: #0ee;
    img {
      width: 100%;
      height: 100%;
      background-color: #eee;
      border-radius: 50%;
    }
  }
  .btns {
    display: flex;
    justify-content: flex-end;
  }
  .login_form {
    position: absolute;
    bottom: 0%;
    width: 100%;
    padding: 0 10px;
    box-sizing: border-box;
  }
  .title {
    top: 20px;
    text-align: center;
  }
}
.identity {
  background-color: rgb(158, 34, 34);
}
.bodyImg {
  top: 10%;
  height: 100%;
  width: 100%;
}
/deep/input::-webkit-input-placeholder {
  color: #17a1e5;
  font-size: 15px;
}
</style>
