<template>
  <el-form
    ref="RefereeFormRef"
    :model="RefereeForm"
    label-width="35%"
    class="Form"
    :rules="RefereeRules"
  >
    <div class="wrap">
      <h1>添加教练员信息</h1>
    </div>

    <el-form-item label="所在队伍" class="teamId" prop="teamId">
      <el-input
        v-model="RefereeForm.teamId"
        placeholder="请输入队伍Id"
        class="teamIdInput"
        @change="teamId"
      ></el-input>
    </el-form-item>

    <el-form-item label="教练姓名" class="name" prop="name">
      <el-input
        v-model="RefereeForm.name"
        placeholder="请输入姓名"
        class="nameInput"
      ></el-input>
    </el-form-item>

    <el-form-item label="教练员身份证" class="identifier" prop="identifier">
      <el-input
        v-model="RefereeForm.identifier"
        placeholder="请输入身份证"
        class="identifierInput"
        @change="identifier"
      ></el-input>
    </el-form-item>

    <el-form-item label="教练员电话号码" class="phone" prop="phone">
      <el-input
        v-model="RefereeForm.phone"
        placeholder="请输入电话号码"
        class="phoneInput"
        @change="phone"
      ></el-input>
    </el-form-item>

    <el-form-item label="性别" class="gender" prop="gender">
      <el-select v-model="RefereeForm.gender" placeholder="请选择性别">
        <el-option label="男" value="m"></el-option>
        <el-option label="女" value="f"></el-option>
      </el-select>
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
      RefereeForm: {
        teamId: "",
        name: "",
        identifier: "",
        phone: "",
        gender: "",
      },
      id: "",
      //验证规则
      RefereeRules: {
        teamId: [
          { required: true, message: "请输入队伍Id", trigger: "blur" },
          { min: 6, max: 6, message: "长度为6个字符", trihger: "blur" },
        ],
        name: [
          { required: true, message: "请输入姓名", trigger: "blur" },
          { min: 2, max: 7, message: "长度在2 ~ 7个字符", trihger: "blur" },
        ],
        identifier: [
          { required: true, message: "请输入教练员身份证", trigger: "blur" },
          { min: 18, max: 18, message: "长度为18个数字", trihger: "blur" },
        ],
        phone: [
          { required: true, message: "请输入教练员电话号码", trigger: "blur" },
          { min: 11, max: 11, message: "长度为11个数字", trihger: "blur" },
        ],
        gender: [{ required: true, message: "请选择性别", trigger: "change" }],
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
    teamId: function () {
      if (!/^\d{6}$/.test(this.RefereeForm.teamId)) {
        this.$message({
          duration: 800,
          message: "请输入长度为6位的Id",
          type: "error",
        });
        this.RefereeForm.teamId = "";
        return;
      }
      //查询队伍是否存在
      this.$http
        .get("gymnastics/team/getbyid/" + this.RefereeForm.teamId)
        .then((res) => {
          if (!res.data) {
            this.$message({
              duration: 600,
              message: "队伍不存在，请重新输入",
              type: "error",
            });
            this.RefereeForm.teamId = "";
            return;
          }
        });
    },
    identifier: function () {
      if (
        !/^([1-6][1-9]|50)\d{4}(18|19|20)\d{2}((0[1-9])|10|11|12)(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/.test(
          this.RefereeForm.identifier
        )
      ) {
        this.$message({
          duration: 800,
          message: "请输入正确的身份证",
          type: "error",
        });
        this.RefereeForm.identifier = "";
        return;
      }
    },
    phone: function () {
      if (!/^1[345789]\d{9}$/.test(this.RefereeForm.phone)) {
        this.$message({
          duration: 800,
          message: "请输入正确的手机号",
          type: "error",
        });
        this.RefereeForm.phone = "";
        return;
      }
    },
    onSubmit() {
      this.$refs.RefereeFormRef.validate(async (valid) => {
        //验证失败
        if (!valid) return;
        //新的教练
        var coach = {
          coachName: this.RefereeForm.name,
          coachIdentifier: this.RefereeForm.identifier,
          coachPhone: this.RefereeForm.phone,
          coachGender: this.RefereeForm.gender,
        };
        //插入新的教练到教练表
        await this.$http.post("gymnastics/coach", coach).then((res) => {
          if (res.data) {
            this.$message({
              duration: 600,
              message: "添加成功！",
              type: "success",
            });
            //更新教练数量
            this.uploadCoachNum(this.RefereeForm.teamId);
            //更新队伍教练表
            this.uploadTeamCoach(this.RefereeForm.teamId);
            //更新裁判表
            this.uploadReferee(this.RefereeForm.teamId);
            this.$router.go(-1);
          } else {
            this.$message({
              duration: 600,
              message: "添加失败",
              type: "error",
            });
          }
        });
      });
    },
    uploadCoachNum(teamId) {
      //查询原来的team
      this.$http.get("gymnastics/team/getbyid/" + teamId).then((res) => {
        if (!res.data) {
          this.$message({
            duration: 600,
            message: "队伍不存在,教练添加失败",
            type: "error",
          });
        } else {
          //更新教练数量
          var team = {
            teamId: res.data.teamId,
            teamName: res.data.teamName,
            registerYear: res.data.registerYear,
            leaderId: res.data.leaderId,
            doctorId: res.data.doctorId,
            coachCount: ++res.data.coachCount,
          };
          this.$http.put("gymnastics/team", team).then((res) => {});
        }
      });
    },
    //更新队伍、教练表
    uploadTeamCoach(teamId) {
      this.$http
        .get("gymnastics/coach/getbyidentify/" + this.RefereeForm.identifier)
        .then((coachres) => {
          if (coachres.data) {
            //更新教练数量
            var coach = {
              teamId: teamId,
              coachId: coachres.data.coachId,
            };
            this.$http.post("gymnastics/team-coach", coach).then((res) => {});
          }
        });
    },
    uploadReferee(teamId) {
      //增加裁判
      this.$http
        .get("gymnastics/team-login/getbyteamid/" + teamId)
        .then((refereeres) => {
          var referee = {
            refereeName: this.RefereeForm.name,
            refereeIdentifier: this.RefereeForm.identifier,
            refereePhone: this.RefereeForm.phone,
            refereeLoginAccount: refereeres.data.teamAccount,
          };

          this.$http.post("gymnastics/referee/", referee).then((res) => {});
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
.identifier {
  height: 8%;
}
.identifierInput {
  width: 60%;
}
.phone {
  height: 8%;
}
.phoneInput {
  width: 60%;
}
.gender {
  height: 8%;
}
</style>