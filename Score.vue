<template>
  <el-form
    ref="ScoreFormRef"
    :model="ScoreForm"
    label-width="35%"
    class="ScoreForm"
    :rules="ScoreFormRules"
  >
    <div class="wrap">
      <h1>裁判打分环节</h1>
    </div>

    <el-form-item label="比赛项目及组别" class="gameId" prop="gameId">
      <el-select v-model="ScoreForm.gameId" placeholder="请选择比赛项目及组别">
        <el-option
          v-for="item in gameIdList"
          :value="item.gameId"
          :label="item.gameNameGroup"
          :key="item.gameId"
        ></el-option>
      </el-select>
    </el-form-item>

    <el-form-item
      label="运动员ID"
      class="athleteId"
      prop="athleteId"
      @change="athleteId"
    >
      <el-select
        v-model="ScoreForm.athleteId"
        placeholder="请选择运动员进行打分"
      >
        <el-option
          v-for="item in athleteIdList"
          :value="item.athleteId"
          :label="item.athleteNameGroup"
          :key="item.athleteId"
        ></el-option>
      </el-select>
    </el-form-item>

    <el-form-item label="选手分数" class="score" prop="score">
      <el-input
        v-model="ScoreForm.score"
        placeholder="请输入该选手的分数"
        class="scoreInput"
        @change="score"
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
      //裁判id
      refereeId: "",
      //赛事Id
      gameId: "",
      //打分表单
      ScoreForm: {
        gameId: "",
        athleteId: "",
        score: "",
      },
      gameIdList: [{ gameId: "", gameNameGroup: "" }],
      athleteIdList: [{ athleteId: "", athleteNameGroup: "" }],
      //验证规则
      ScoreFormRules: {
        gameId: [{ required: true, message: "请选择比赛ID", trigger: "blur" }],
        athleteId: [
          { required: true, message: "请选择运动员ID", trigger: "blur" },
        ],
        score: [{ required: true, message: "请输入选手得分", trigger: "blur" }],
      },
    };
  },
  created() {
    //接收refereeId
    this.refereeId = this.$route.query.refereeId;
    console.log(this.refereeId);
    //插入比赛项目及组别
    this.$http
      .get("gymnastics/game-referee/getbyrefereeid/" + this.refereeId)
      .then((res) => {
        this.gameId = res.data.gameId;
        this.$http.get("gymnastics/game/getbyid/" + this.gameId).then((res) => {
          var game = {
            gameId: res.data.gameId,
            gameNameGroup: res.data.gameName + "第" + res.data.gameGroup + "组",
          };
          this.gameIdList.push(game);
        });
      });
    this.gameIdList.splice(0, 1);

    //插入运动员信息
    this.$http
      .get("gymnastics/game-referee/getbyrefereeid/" + this.refereeId)
      .then((res) => {
        this.gameId = res.data.gameId;
        this.$http
          .get("gymnastics/competition/getbygameid/" + this.gameId)
          .then((res) => {
            res.data.forEach((element) => {
              this.$http
                .get("gymnastics/athlete/getbyid/" + element.athleteId)
                .then((res) => {
                  var athlete = {
                    athleteId: res.data.athleteId,
                    athleteNameGroup:
                      res.data.athleteCode + "号选手" + res.data.athleteName,
                  };
                  this.athleteIdList.push(athlete);
                });
            });
          });
      });
    this.athleteIdList.splice(0, 1);
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
    athleteId: function () {
      if (this.ScoreForm.gameId == []) {
        this.$message({
          duration: 1000,
          message: "请先选择比赛项目及组别",
          type: "error",
        });
        this.ScoreForm.athleteId = [];
        return;
      }
    },
    score: function () {
      if (!/^\d{1,3}$/.test(this.ScoreForm.score)) {
        this.$message({
          duration: 1000,
          message: "请输入正确格式的分数",
          type: "error",
        });
        this.ScoreForm.score = "";
        return;
      }
      if (
        parseInt(this.ScoreForm.score) < 0 ||
        parseInt(this.ScoreForm.score) > 100
      ) {
        this.$message({
          duration: 1000,
          message: "你输入的分数有误，请重新输入",
          type: "error",
        });
        this.ScoreForm.score = "";
        return;
      }
    },
    onSubmit() {
      this.$refs.ScoreFormRef.validate(async (valid) => {
        //验证失败
        if (!valid) return;
        console.log(this.ScoreForm.gameId);
        console.log(this.ScoreForm.athleteId);

        this.$http
          .get("gymnastics/competition/getbygameid/" + this.ScoreForm.gameId)
          .then((res) => {
            console.log(res);
            res.data.forEach((element) => {
              if (element.athleteId == this.ScoreForm.athleteId) {
                //更新分数
                var game = {
                  competitionId: element.competitionId,
                  gameId: this.ScoreForm.gameId,
                  athleteId: this.ScoreForm.athleteId,
                  athleteScore: this.ScoreForm.score,
                };
                this.$http
                  .put("gymnastics/competition/", game)
                  .then((res) => {
                    console.log(res);
                    if (res.data) {
                      this.$message({
                        duration: 1000,
                        message: "评分成功",
                        type: "success",
                      });
                      this.ScoreForm.gameId = "";
                      this.ScoreForm.athleteId = "";
                      this.ScoreForm.score = "";
                    } else {
                      this.$message({
                        duration: 1000,
                        message: "评分失败",
                        type: "error",
                      });
                    }
                  })
                  .catch((err) => {
                    this.$message.error(err);
                  });
              }
            });
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
.ScoreForm {
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
.gameId {
  height: 10%;
}

.athleteId {
  height: 10%;
}
.score {
  height: 10%;
}
.scoreInput {
  width: 38%;
}
</style>