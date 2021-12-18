<template>
  <div>
    <el-form label-width="40%" class="Form" id="DoctorForm">
      <div class="wrap">
        <h1>队医信息</h1>
      </div>
      <el-form-item label="队伍" class="TeamLabel">
        <span class="Team">{{ team }}</span>
      </el-form-item>
      <el-form-item label="姓名" class="NameLabel">
        <span class="Name">{{ name }}</span>
      </el-form-item>
      <el-form-item label="DoctorID" class="DoctorIDLabel">
        <span class="DoctorID">{{ doctorId }}</span>
      </el-form-item>
      <el-form-item label="身份证号码" class="IdentifierLabel">
        <span class="Identifier">{{ identifier }}</span>
      </el-form-item>
      <el-form-item label="电话" class="phoneLabel">
        <span class="Phone">{{ phone }}</span>
      </el-form-item>

      <div class="button">
        <el-button type="" @click="back" icon="el-icon-s-promotion"
          >返回</el-button
        >
      </div>
    </el-form>
  </div>
</template>
<script>
import routers from "../../router";
export default {
  data() {
    return {
      //队伍ID
      teamId: "",
      //队医信息
      team: "",
      name: "",
      doctorId: "",
      identifier: "",
      phone: "",
    };
  },
  created() {
    this.teamId = this.$route.query.teamId;
    //根据teamID查找
    this.$http
      .get("gymnastics/team/getbyid/" + this.teamId)
      .then((res) => {
        //队伍名称加ID
        this.team = res.data.teamName + "(" + res.data.teamId + ")";

        this.$http
          .get("gymnastics/doctor/getbyid/" + res.data.doctorId)
          .then((doctorres) => {
            console.log(doctorres);
            this.name = doctorres.data.doctorName;
            this.doctorId = doctorres.data.doctorId;
            this.identifier = doctorres.data.doctorIdentifier;
            this.phone = doctorres.data.doctorPhone;
          });
      })
      .catch((err) => {
        this.$message.error(err);
      });
  },
  methods: {
    navigate() {
      routers.go(-1);
    },
    back() {
      this.$router.push("/home");
    },
  },
};
</script>
<style lang="less" scoped>
.wrap {
  height: 20%;
  width: 90%;
  display: table;
  text-align: center;
}
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
.button {
  display: block;
  position: absolute;
  width: 50%;
  left: 42%;
  top: 80%;
}
.TeamLabel {
  height: 8%;
}
.Team {
  width: 50%;
  font-size: 20px;
}
.NameLabel {
  height: 8%;
}
.Name {
  width: 50%;
  font-size: 20px;
}
.DoctorIDLabel {
  height: 8%;
}
.DoctorID {
  width: 50%;
  font-size: 20px;
}
.PhoneLabel {
  height: 8%;
}
.Phone {
  width: 50%;
  font-size: 20px;
}
.IdentifierLabel {
  height: 8%;
}
.Identifier {
  width: 50%;
  font-size: 20px;
}
</style>