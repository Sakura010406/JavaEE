<template>
  <div>
    <!-- 比赛表格 -->
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
        prop="gameId"
        label="比赛ID"
        width="150"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="gameName"
        label="赛事类型"
        width="150"
        align="center"
      >
      </el-table-column>
      <el-table-column
        prop="gameYear"
        label="比赛年份"
        width="150"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="gameGroup"
        label="比赛组别"
        width="150"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="gameStage"
        label="比赛阶段"
        width="150"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="chiefRefereeId"
        label="主裁判ID"
        width="150"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="refereeCount"
        label="裁判数量"
        width="150"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="finishedCode"
        label="是否结束"
        width="150"
        align="center"
      ></el-table-column>

      <el-table-column fixed="right" label="操作" width="150" align="center">
        <template slot-scope="scope">
          <el-button
            @click.native.prevent="
              startGame(scope.$index, tableData, scope.row)
            "
            type="text"
            size="small"
          >
            开始比赛
          </el-button>
        </template>
      </el-table-column>
    </el-table>
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
          gameId: "",
          gameName: "",
          gameYear: "",
          gameGroup: "",
          gameStage: "",
          chiefRefereeId: "",
          refereeCount: "",
          finishedCode: "",
        },
      ],
      //比赛分页参数
      total: 0,
      currentPage: 1,
      pageSize: 9,
      loading: false,
    };
  },
  created() {
    this.select_Game();
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
    //获取比赛总数
    totalData() {
      this.$http.get("gymnastics/game/getbygamestage/" + 1).then((res) => {
        this.total = res.data.legth;
      });
    },
    //分页查询
    select_Game() {
      this.totalData();
      this.loading = true;
      this.$http
        .get(
          "gymnastics/game/getbypage/" +
            this.currentPage.toString() +
            "/" +
            this.pageSize.toString()
        )
        .then((res) => {
          this.loading = false;
          console.log(res);
          //删除空白行
          this.tableData.splice(0, 1);
          res.data.records.forEach((element) => {
            if (element.gameStage == 1) {
              var game = {
                gameId: element.gameId,
                gameName: element.gameName,
                gameYear: element.gameYear,
                gameGroup: element.gameGroup,
                gameStage: "决赛",
                chiefRefereeId: element.gameChiefRefereeId,
                refereeCount: element.gameRefereeCount,
                finishedCode: element.gameFinished == 0 ? "未完成" : "已结束",
              };
              this.tableData.push(game);
            }
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
      this.select_Game();
    },
    //调整size
    handleSizeChange(val) {
      this.pageSize = val;
    },
    startGame(index, rows, row) {
     
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
          "比赛ID",
          "赛事类型",
          "比赛年份",
          "比赛组别",
          "比赛阶段",
          "主裁判ID",
          "教练员数量",
          "是否结束",
        ];
        const filterVal = [
          "gameId",
          "gameName",
          "gameYear",
          "gameGroup",
          "gameStage",
          "chiefRefereeId",
          "refereeCount",
          "finishedCode",
        ];
        const list = this.tableData; //把data里的tableData存到list
        const data = this.formatJson(filterVal, list);
        export_json_to_excel(tHeader, data, "决赛赛事表");
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
</style>