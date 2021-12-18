<template>
  <el-form
    ref="GameFormRef"
    :model="GameForm"
    label-width="35%"
    class="Form"
    :rules="GameRules"
  >
    <div class="wrap">
      <h1>添加比赛信息</h1>
    </div>

    <el-form-item label="赛事类型" class="gameName" prop="gameName">
      <el-select v-model="GameForm.gameName" placeholder="请选择比赛类型">
        <el-option label="男子单杠" value="男子单杠"></el-option>
        <el-option label="男子双杠" value="男子双杠"></el-option>
        <el-option label="男子吊环" value="男子吊环"></el-option>
        <el-option label="男子跳马" value="男子跳马"></el-option>
        <el-option label="女子跳马" value="女子跳马"></el-option>
        <el-option label="女子高低杠" value="女子高低杠"></el-option>
        <el-option label="女子平衡木" value="女子平衡木"></el-option>
        <el-option label="男子自由体操" value="男子自由体操"></el-option>
        <el-option label="女子自由体操" value="女子自由体操"></el-option>
        <el-option label="男子鞍马" value="男子鞍马"></el-option>
        <el-option label="男子蹦床" value="男子蹦床"></el-option>
        <el-option label="女子蹦床" value="女子蹦床"></el-option>
      </el-select>
    </el-form-item>

    <el-form-item label="比赛年份" class="gameYear" prop="gameYear">
      <el-col :span="11">
        <el-date-picker
          type="year"
          placeholder="请选择比赛年份"
          v-model="GameForm.gameYear"
          style="width: 100%"
          value-format="yyyy"
          format="yyyy"
        ></el-date-picker>
      </el-col>
    </el-form-item>

    <el-form-item label="比赛组别" class="gameGroup" prop="gameGroup">
      <el-input
        v-model="GameForm.gameGroup"
        placeholder="请设置比赛组别"
        class="gameGroupInput"
        @change="gameGroup"
      ></el-input>
    </el-form-item>

    <el-form-item label="比赛阶段" class="gameStage" prop="gameStage">
      <el-select v-model="GameForm.gameStage" placeholder="请选择比赛阶段">
        <el-option label="初赛" value="0"></el-option>
        <el-option label="决赛" value="1"></el-option>
      </el-select>
    </el-form-item>

    <el-form-item label="主裁判ID" class="chiefRefereeId" prop="chiefRefereeId">
      <el-input
        v-model="GameForm.chiefRefereeId"
        placeholder="请输入主裁判ID"
        class="chiefRefereeIdInput"
        @change="chiefRefereeId"
      ></el-input>
    </el-form-item>

    <el-form-item label="裁判数量" class="refereeCount" prop="refereeCount">
      <el-input
        v-model="GameForm.refereeCount"
        placeholder="请输入裁判数量"
        class="refereeCountInput"
        @change="refereeCount"
      ></el-input>
    </el-form-item>

    <el-form-item label="完成情况" class="finishedCode" prop="finishedCode">
      <el-input
        v-model="GameForm.finishedCode"
        placeholder=""
        class="finishedCodeInput"
        :disabled="true"
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
      GameForm: {
        gameName: "",
        gameYear: "",
        gameGroup: "",
        gameStage: "",
        chiefRefereeId: "",
        refereeCount: "",
        finishedCode: 0,
      },

      //验证规则
      GameRules: {
        gameName: [
          { required: true, message: "请选择比赛类型", trigger: "change" },
        ],
        gameYear: [
          { required: true, message: "请选择比赛年份", trigger: "change" },
        ],
        gameGroup: [
          { required: true, message: "请设置比赛组别", trigger: "blur" },
          { min: 0, max: 3, message: "长度为1-3个字符", trihger: "blur" },
        ],
        gameStage: [
          { required: true, message: "请选择比赛阶段", trigger: "change" },
        ],
        chiefRefereeId: [
          { required: true, message: "请输入主裁判ID", trigger: "blur" },
          { min: 0, max: 6, message: "长度为1-6个字符", trihger: "blur" },
        ],
        refereeCount: [
          { required: true, message: "请输入主裁判ID", trigger: "blur" },
          { min: 0, max: 1, message: "长度为1个字符", trihger: "blur" },
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
    gameGroup: function () {
      if (!/^\d{1,3}$/.test(this.GameForm.gameGroup)) {
        this.$message({
          duration: 1000,
          message: "请输入长度为1-3位的组别",
          type: "error",
        });
        this.GameForm.gameGroup = "";
        return;
      }
      //查询该组别是否存在
      this.$http
        .get("gymnastics/game/getbygamename/" + this.GameForm.gameName)
        .then((res) => {
          res.data.forEach((element) => {
            if (element.gameGroup == this.GameForm.gameGroup) {
              this.$message({
                duration: 1000,
                message: "该组别已存在，请重新输入",
                type: "error",
              });
              this.GameForm.gameGroup = "";
              return;
            }
          });
        });
    },
    chiefRefereeId: function () {
      //查询该裁判是否存在
      this.$http
        .get("gymnastics/referee/getbyid/" + this.GameForm.chiefRefereeId)
        .then((res) => {
          if (!res.data) {
            this.$message({
              duration: 1000,
              message: "不存在该裁判，请重新输入",
              type: "error",
            });
            this.GameForm.chiefRefereeId = "";
            return;
          }
        });
    },
    refereeCount: function () {
      if (
        !/^\d{1,3}$/.test(this.GameForm.refereeCount) ||
        this.GameForm.refereeCount.toString() == "0"
      ) {
        this.$message({
          duration: 1000,
          message: "请输入长度为1-3位且不为零的裁判数量",
          type: "error",
        });
        this.GameForm.refereeCount = "";
        return;
      }
    },
    onSubmit() {
      this.$refs.GameFormRef.validate(async (valid) => {
        //验证失败
        if (!valid) return;

        //新的比赛
        var game = {
          gameName: this.GameForm.gameName,
          gameYear: this.GameForm.gameYear,
          gameGroup: this.GameForm.gameGroup,
          gameStage: this.GameForm.gameStage,
          gameChiefRefereeId: this.GameForm.chiefRefereeId,
          gameRefereeCount: this.GameForm.refereeCount,
          gameFinished: this.GameForm.finishedCode,
        };
        //插入新的运动员
        await this.$http.post("gymnastics/game", game).then((res) => {
          if (res.data) {
            this.$message({
              duration: 1000,
              message: "添加成功！",
              type: "success",
            });
            this.$router.go(-1);
          } else {
            this.$message({
              duration: 1000,
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
.gameName {
  height: 8%;
}
.gameYear {
  height: 8%;
}
.gameGroup {
  height: 8%;
}
.gameGroupInput {
  width: 50%;
}
.gameStage {
  height: 8%;
}
.chiefRefereeId {
  height: 8%;
}
.chiefRefereeIdInput {
  width: 50%;
}
.refereeCount {
  height: 8%;
}
.refereeCountInput {
  width: 50%;
}
.finishedCode {
  height: 8%;
}
.finishedCodeInput {
  width: 60%;
}
</style>