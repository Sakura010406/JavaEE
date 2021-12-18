<template>
  <el-form
    ref="DoctorFormRef"
    :model="DoctorForm"
    label-width="35%"
    class="Form"
    :rules="DoctorRules"
  >
    <div class="wrap">
      <h1>添加队医信息</h1>
    </div>

    <el-form-item label="所在队伍" class="teamId" prop="teamId">
      <el-input
        v-model="DoctorForm.teamId"
        placeholder="请输入队伍Id"
        class="teamIdInput"
        @change="teamId"
      ></el-input>
    </el-form-item>

    <el-form-item label="队医姓名" class="name" prop="name">
      <el-input
        v-model="DoctorForm.name"
        placeholder="请输入姓名"
        class="nameInput"
      ></el-input>
    </el-form-item>

    <el-form-item label="队医身份证" class="identifier" prop="identifier">
      <el-input
        v-model="DoctorForm.identifier"
        placeholder="请输入身份证"
        class="identifierInput"
        @change="identifier"
      ></el-input>
    </el-form-item>

    <el-form-item label="队医电话号码" class="phone" prop="phone">
      <el-input
        v-model="DoctorForm.phone"
        placeholder="请输入电话号码"
        class="phoneInput"
        @change="phone"
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
      DoctorForm: {
        teamId: "",
        name: "",
        identifier: "",
        phone: "",
      },
      id: "",
      //验证规则
      DoctorRules: {
        teamId: [
          { required: true, message: "请输入队伍Id", trigger: "blur" },
          { min: 6, max: 6, message: "长度为6个字符", trihger: "blur" },
        ],
        name: [
          { required: true, message: "请输入姓名", trigger: "blur" },
          { min: 2, max: 7, message: "长度在2 ~ 7个字符", trihger: "blur" },
        ],
        identifier: [
          { required: true, message: "请输入队医身份证", trigger: "blur" },
          { min: 18, max: 18, message: "长度为18个数字", trihger: "blur" },
        ],
        phone: [
          { required: true, message: "请输入队医电话号码", trigger: "blur" },
          { min: 11, max: 11, message: "长度为11个数字", trihger: "blur" },
        ],
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
      if (!/^\d{6}$/.test(this.DoctorForm.teamId)) {
        this.$message({
          duration: 1000,
          message: "请输入长度为6位的Id",
          type: "error",
        });
        this.DoctorForm.teamId = "";
        return;
      }
      //查询队伍是否存在
      this.$http
        .get("gymnastics/team/getbyid/" + this.DoctorForm.teamId)
        .then((res) => {
          if (!res.data) {
            this.$message({
              duration: 1000,
              message: "队伍不存在，请重新输入",
              type: "error",
            });
            this.DoctorForm.teamId = "";
            return;
          }
          if (res.data.doctorId != null) {
            this.$message({
              duration: 1000,
              message: "队伍已存在队医,禁止添加",
              type: "error",
            });
            this.exit();
            return;
          }
        });
    },
    identifier: function () {
      if (
        !/^([1-6][1-9]|50)\d{4}(18|19|20)\d{2}((0[1-9])|10|11|12)(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/.test(
          this.DoctorForm.identifier
        )
      ) {
        this.$message({
          duration: 1000,
          message: "请输入正确的身份证",
          type: "error",
        });
        this.DoctorForm.identifier = "";
        return;
      }
    },
    phone: function () {
      if (!/^1[345789]\d{9}$/.test(this.DoctorForm.phone)) {
        this.$message({
          duration: 1000,
          message: "请输入正确的手机号",
          type: "error",
        });
        this.DoctorForm.phone = "";
        return;
      }
    },
    onSubmit() {
      this.$refs.DoctorFormRef.validate(async (valid) => {
        //验证失败
        if (!valid) return;
        //新的队医
        var doctor = {
          doctorName: this.DoctorForm.name,
          doctorIdentifier: this.DoctorForm.identifier,
          doctorPhone: this.DoctorForm.phone,
        };
        // //插入新的医生到队医表
        await this.$http.post("gymnastics/doctor", doctor).then((res) => {
          if (res.data) {
            this.$message({
              duration: 600,
              message: "添加成功！",
              type: "success",
            });

            this.$router.go(-1);
          } else {
            this.$message({
              duration: 600,
              message: "添加失败",
              type: "error",
            });
          }
        });
        //更新team表中的doctorId
        this.uploadDOctorID(this.DoctorForm.identifier);
      });
    },
    uploadDOctorID(doctorIdentifier) {
      this.$http
        .get("gymnastics/doctor/getbyidentify/" + doctorIdentifier)
        .then((res) => {
          var team = {
            teamId: this.DoctorForm.teamId,
            doctorId: res.data.doctorId,
          };
          this.$http.put("gymnastics/team", team).then((res) => {});
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
</style>