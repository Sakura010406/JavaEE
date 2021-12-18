<template>
  <el-form
    ref="AthleteFormRef"
    :model="AthleteForm"
    label-width="35%"
    class="Form"
    :rules="AthleteRules"
  >
    <div class="wrap">
      <h1>添加运动员信息</h1>
    </div>

    <el-form-item label="所在队伍" class="teamId" prop="teamId">
      <el-input
        v-model="AthleteForm.teamId"
        placeholder="请输入队伍id"
        class="teamIdInput"
        @change="teamId"
      ></el-input>
    </el-form-item>
    <el-form-item label="运动员姓名" class="name" prop="name">
      <el-input
        v-model="AthleteForm.name"
        placeholder="请输入姓名"
        class="nameInput"
      ></el-input>
    </el-form-item>

    <el-form-item label="性别" class="gender" prop="gender">
      <el-select v-model="AthleteForm.gender" placeholder="请选择性别">
        <el-option label="男" value="m"></el-option>
        <el-option label="女" value="f"></el-option>
      </el-select>
    </el-form-item>

    <el-form-item label="运动员身份证" class="identifier" prop="identifier">
      <el-input
        v-model="AthleteForm.identifier"
        placeholder="请输入身份证"
        class="identifierInput"
        @change="identifier"
      ></el-input>
    </el-form-item>

    <el-form-item label="出生日期" class="date" prop="birth">
      <el-col :span="11">
        <el-date-picker
          type="date"
          placeholder="出生日期"
          v-model="AthleteForm.birth"
          style="width: 100%"
          value-format="yyyy-MM-dd"
          format="yyyy 年 MM 月 dd 日"
        ></el-date-picker>
      </el-col>
    </el-form-item>
    <el-form-item label="运动员编号" class="athleteCode" prop="athleteCode">
      <el-input
        v-model="AthleteForm.athleteCode"
        placeholder="生成运动员编号"
        class="athleteCodeInput"
        :disabled="true"
      ></el-input>
      <el-button type="primary" @click="createCode">生成</el-button>
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
      AthleteForm: {
        athleteCode: "",
        teamId: "",
        name: "",
        gender: "",
        identifier: "",
        birth: "",
      },
      lastMaleAthleteCode: "",
      lastFemaleAthleteCode: "",

      //验证规则
      AthleteRules: {
        teamId: [
          { required: true, message: "请输入队伍id", trigger: "blur" },
          { min: 6, max: 6, message: "长度为6个数字", trihger: "blur" },
        ],
        name: [
          { required: true, message: "请输入姓名", trigger: "blur" },
          { min: 2, max: 7, message: "长度在2 ~ 7个字符", trihger: "blur" },
        ],
        gender: [{ required: true, message: "请选择性别", trigger: "change" }],
        identifier: [
          { required: true, message: "请输入运动员身份证", trigger: "blur" },
          { min: 18, max: 18, message: "长度为18个数字", trihger: "blur" },
        ],
        birth: [{ required: true, message: "请选择出生日期", trigger: "blur" }],
      },
    };
  },
  created() {
    this.keyupSubmit();
  },
  methods: {
    createCode() {
      //查找运动员编号
      if (this.AthleteForm.gender == []) {
        this.$message({
          duration: 600,
          message: "请先选择性别!",
          type: "error",
        });
        return;
      } else {
        this.$http.get("gymnastics/athlete-code/getbyid/" + 1).then((res) => {
          if (this.AthleteForm.gender == "m") {
            this.AthleteForm.athleteCode =
              parseInt(res.data.lastMaleAthleteCode) + 2;
            this.lastMaleAthleteCode = this.AthleteForm.athleteCode;
            this.lastFemaleAthleteCode = res.data.lastFemaleAthleteCode;
            return;
          } else {
            this.AthleteForm.athleteCode =
              parseInt(res.data.lastFemaleAthleteCode) + 2;
            this.lastMaleAthleteCode = res.data.lastMaleAthleteCode;
            this.lastFemaleAthleteCode = this.AthleteForm.athleteCode;
            return;
          }
        });
      }
    },
    resetCode(lastMaleCode, lastFemaleCode) {
      //更新第一条记录   新的记录
      var athleteCode = {
        id: 1,
        lastMaleAthleteCode: lastMaleCode,
        lastFemaleAthleteCode: lastFemaleCode,
      };
      this.$http.put("gymnastics/athlete-code", athleteCode).then((res) => {});
    },
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
      if (!/^\d{6}$/.test(this.AthleteForm.teamId)) {
        this.$message({
          duration: 800,
          message: "请输入长度为6位的Id",
          type: "error",
        });
        this.AthleteForm.teamId = "";
        return;
      }
      //查询队伍是否存在
      this.$http
        .get("gymnastics/team/getbyid/" + this.AthleteForm.teamId)
        .then((res) => {
          if (!res.data) {
            this.$message({
              duration: 600,
              message: "队伍不存在，请重新输入",
              type: "error",
            });
            this.AthleteForm.teamId = "";
            return;
          }
        });
    },
    identifier: function () {
      if (
        !/^([1-6][1-9]|50)\d{4}(18|19|20)\d{2}((0[1-9])|10|11|12)(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/.test(
          this.AthleteForm.identifier
        )
      ) {
        this.$message({
          duration: 800,
          message: "请输入正确的身份证！",
          type: "error",
        });
        this.AthleteForm.identifier = "";
        return;
      }
    },
    onSubmit() {
      this.$refs.AthleteFormRef.validate(async (valid) => {
        //验证失败
        if (!valid) return;

        //新的运动员
        var athlete = {
          athleteCode: this.AthleteForm.athleteCode,
          athleteName: this.AthleteForm.name,
          teamId: this.AthleteForm.teamId,
          athleteBirthyear: this.AthleteForm.birth,
          athleteIdentifier: this.AthleteForm.identifier,
          athleteGender: this.AthleteForm.gender,
        };
        //插入新的运动员
        await this.$http.post("gymnastics/athlete", athlete).then((res) => {
          if (res.data) {
            this.$message({
              duration: 600,
              message: "添加成功！",
              type: "success",
            });
            //重置运动员Code
            this.resetCode(
              this.lastMaleAthleteCode,
              this.lastFemaleAthleteCode
            );
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
  width: 50%;
}
.athleteCode {
  height: 8%;
}
.athleteCodeInput {
  width: 50%;
}
.name {
  height: 8%;
}
.nameInput {
  width: 50%;
}
.gender {
  height: 8%;
}
.identifier {
  height: 8%;
}
.identifierInput {
  width: 60%;
}
</style>