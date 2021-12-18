<template>
  <div>
    <!-- 运动员成绩 -->
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
        prop="CompetitionID"
        label="COMPETITION---ID"
        width="300"
        align="center"
      ></el-table-column>

      <el-table-column prop="GameID" label="赛事ID" width="300" align="center">
      </el-table-column>

      <el-table-column
        prop="AthleteID"
        label="运动员ID"
        width="300"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="AthleteScore"
        label="运动员分数"
        width="300"
        align="center"
      ></el-table-column>
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
          CompetitionID: "",
          GameID: "",
          AthleteID: "",
          AthleteScore: "",
        },
      ],
      total: 0,
      currentPage: 1,
      pageSize: 9,
      loading: false,
    };
  },
  created() {
    this.select_Score();
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
    //获取教练总数
    totalData() {
      this.$http.get("gymnastics/competition/findAll").then((res) => {
        this.total = res.data;
      });
    },
    //分页查询
    select_Score() {
      this.totalData();
      this.loading = true;
      this.$http
        .get(
          "gymnastics/competition/getbypage/" +
            this.currentPage.toString() +
            "/" +
            this.pageSize.toString()
        )
        .then((res) => {
          this.loading = false;
          //删除空白行
          this.tableData.splice(0, 1);
          res.data.records.forEach((element) => {
            var athleteScore = {
              CompetitionID: element.competitionId,
              GameID: element.gameId,
              AthleteID: element.athleteId,
              AthleteScore: element.athleteScore,
            };
            this.tableData.push(athleteScore);
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
      this.select_Score();
    },
    //调整size
    handleSizeChange(val) {
      this.pageSize = val;
    },
    //导出教练excel
    update() {
      this.export2Excel();
    },
    //导出excel方法
    export2Excel() {
      require.ensure([], () => {
        const { export_json_to_excel } = require("../../excel/Export2Excel.js");
        //设置Excel的表格第一行的标题
        const tHeader = ["竞赛ID", "比赛ID", "运动员ID", "运动员分数"];
        const filterVal = [
          "CompetitionID",
          "GameID",
          "AthleteID",
          "phone",
          "AthleteScore",
        ];
        const list = this.tableData; //把data里的tableData存到list
        const data = this.formatJson(filterVal, list);
        export_json_to_excel(tHeader, data, "运动员分数表");
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