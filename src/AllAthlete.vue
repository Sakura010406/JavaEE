<template>
  <div>
    <!-- 运动员表格 -->
    <el-table
      class="table"
      :data="tableData"
      v-loading="loading"
      element-loading-text="拼命加载中"
      element-loading-spinner="iconfont icon-jiazai"
      element-loading-background="rgba(0, 0, 0, 0.8)"
    >
      <el-table-column
        fixed
        prop="id"
        label="运动员ID"
        width="150"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="code"
        label="运动员编号"
        width="150"
        align="center"
      >
      </el-table-column>
      <el-table-column prop="teamID" label="队伍ID" width="150" align="center">
      </el-table-column>
      <el-table-column prop="name" label="姓名" width="200" align="center">
      </el-table-column>
      <el-table-column
        prop="gender"
        label="性别"
        width="150"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="identifier"
        label="身份证号码"
        width="300"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="birth"
        label="出生日期"
        width="200"
        align="center"
      ></el-table-column>

      <el-table-column fixed="right" label="操作" width="150" align="center">
        <template slot-scope="scope">
          <el-button
            @click.native.prevent="joinGame(scope.$index, tableData, scope.row)"
            type="text"
            size="small"
          >
            参加比赛
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--参加比赛对话框-->
    <el-dialog title="比赛项目" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="运动员ID" :label-width="formLabelWidth">
          <el-input
            v-model="form.name"
            autocomplete="off"
            :disabled="true"
            class="codeInput"
          ></el-input>
        </el-form-item>
        <el-form-item label="比赛项目及组别" :label-width="formLabelWidth">
          <el-select v-model="form.gameId" placeholder="请选择比赛项目及组别">
            <el-option
              v-for="item in gameIdList"
              :value="item.gameId"
              :label="item.gameNameGroup"
              :key="item.gameId"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="failjoinGame()">取 消</el-button>
        <el-button type="primary" @click="successjoinGame()">确 定</el-button>
      </div>
    </el-dialog>
    <!--分页-->
    <el-pagination
      :total="total"
      :current-page="currentPage"
      :page-size="pageSize"
      :page-sizes="[9, 18, 27, 36]"
      :pager-count="5"
      @current-change="handleCurrentChange"
      @size-change="handleSizeChange"
      background
      layout="total, sizes, prev, pager, next"
      class="page"
    >
    </el-pagination>

    <!-- 导出Excel-->
    <el-button type="" @click="update" icon="el-icon-s-promotion" class="upload"
      >导出Excel</el-button
    >
  </div>
</template>
<script>
import routers from "../../router";
export default {
  data() {
    return {
      tableData: [
        {
          id: "",
          code: "",
          teamID: "",
          name: "",
          gender: "",
          identifier: "",
          birth: "",
        },
      ],
      //队伍
      team: "",
      total: 0,
      currentPage: 1,
      pageSize: 9,
      loading: false,

      //对话框
      dialogTableVisible: false,
      dialogFormVisible: false,
      form: {
        name: "",
        gameId: "",
      },
      formLabelWidth: "120px",
      gameIdList: [{ gameId: "", gameNameGroup: "" }],
    };
  },
  created() {
    this.select_user();
    //初赛
    this.$http.get("gymnastics/game/getbygamestage/" + 0).then((res) => {
      console.log(res);
      res.data.forEach((element) => {
        var game = {
          gameId: element.gameId,
          gameNameGroup: element.gameName + "第" + element.gameGroup + "组",
        };
        this.gameIdList.push(game);
      });
    });
    //决赛
    this.$http.get("gymnastics/game/getbygamestage/" + 1).then((res) => {
      res.data.forEach((element) => {
        var game = {
          gameId: element.gameId,
          gameNameGroup: element.gameName + "第" + element.gameGroup + "组",
        };

        this.gameIdList.push(game);
      });
    });
    this.gameIdList.splice(0, 1);
    //this.keyupSubmit();
  },
  methods: {
    //键盘监视
    keyupSubmit() {
      document.onkeydown = (e) => {
        let _key = window.event.keyCode;
        if (_key === 13) {
          this.query();
        }
      };
    },
    navigate() {
      routers.go(-1);
    },
    //获取运动员总数
    totalData() {
      this.$http.get("gymnastics/athlete/findAll").then((res) => {
        this.total = res.data;
      });
    },
    //分页查询
    select_user() {
      this.totalData();
      this.loading = true;
      this.$http
        .get(
          "gymnastics/athlete/getbypage/" +
            this.currentPage.toString() +
            "/" +
            this.pageSize.toString()
        )
        .then((res) => {
          this.loading = false;
          //删除空白行
          this.tableData.splice(0, 1);
          res.data.records.forEach((element) => {
            var athlete = {
              id: element.athleteId,
              code: element.athleteCode,
              teamID: element.teamId,
              name: element.athleteName,
              gender: element.athleteGender,
              identifier: element.athleteIdentifier,
              birth: element.athleteBirthyear,
            };
            this.tableData.push(athlete);
          });
        })
        .catch((err) => {
          this.loading = false;
          this.$message.error(err);
        });
    },
    //点击下一页
    handleCurrentChange(val) {
      this.currentPage = val;
      this.select_user();
    },
    //调整size
    handleSizeChange(val) {
      this.pageSize = val;
    },
    joinGame(index, rows, row) {
      this.dialogFormVisible = true;
      this.form.name = row.id;
    },
    failjoinGame() {
      this.dialogFormVisible = false;
      this.$message({
        duration: 1000,
        message: "未选择比赛项目，参加比赛失败",
        type: "error",
      });
    },
    successjoinGame() {
      if (this.form.gameId == "") {
        this.$message({
          duration: 1000,
          message: "请选择选择比赛项目",
          type: "error",
        });
        return;
      }
      //查找参赛人数
      console.log(this.form.gameId);
      this.$http
        .get("gymnastics/competition/getbygameid/" + this.form.gameId)
        .then((res) => {
          if (res.data.length < 6) {
            //新的参赛记录
            var competition = {
              gameId: this.form.gameId,
              athleteId: this.form.name,
              athleteScore: 0,
            };
            // //插入参赛记录
            this.$http
              .post("gymnastics/competition", competition)
              .then((res) => {});
            this.dialogFormVisible = false;
            console.log(this.form.name);
            this.$message({
              duration: 1000,
              message: "参加比赛成功",
              type: "success",
            });
            this.form.gameId = [];
          } else {
            this.$message({
              duration: 1000,
              message: "该组人数已达上限,参加比赛失败",
              type: "error",
            });
            this.form.gameId = [];
          }
        });
    },
    //导出运动员excel
    update() {
      this.export2Excel();
    },
    //导出excel方法
    export2Excel() {
      require.ensure([], () => {
        const { export_json_to_excel } = require("../../excel/Export2Excel.js");
        //设置Excel的表格第一行的标题
        const tHeader = [
          "序号",
          "运动员编号",
          "队伍ID",
          "姓名",
          "性别",
          "身份证号码",
          "出生日期",
        ];
        const filterVal = [
          "id",
          "code",
          "teamID",
          "name",
          "gender",
          "identifier",
          "birth",
        ];
        const list = this.tableData; //把data里的tableData存到list
        const data = this.formatJson(filterVal, list);
        export_json_to_excel(tHeader, data, "运动员表");
      });
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map((v) => filterVal.map((j) => v[j]));
    },
  },
};
</script>

<style lang="less"  scoped>
.table {
  display: block;
  position: absolute;
  top: 10%;
  right: 0;
  width: 80%;
  height: 90%;
  bottom: 0;
}
.page {
  display: block;
  position: absolute;
  width: 50%;
  height: 5%;
  right: 5%;
  bottom: 2.5%;
  color: aqua;
}
.upload {
  display: block;
  position: absolute;
  text-align: center;
  color: rgb(27, 21, 21);
  width: 125px;
  height: 40px;
  top: 2%;
  right: 200px;
  background-color: rgb(193, 219, 208);
}
.codeInput {
  width: 80%;
}
</style>