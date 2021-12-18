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
        label="AthleteID"
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

      <el-table-column fixed="right" label="查看" width="150" align="center">
        <template slot-scope="scope">
          <el-button
            @click.native.prevent="
              showDetails(scope.$index, tableData, scope.row)
            "
            type="text"
            size="small"
          >
            查看
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
          id: "",
          code: "",
          name: "",
          gender: "",
          identifier: "",
          birth: "",
        },
      ],
      //队伍Id
      teamId: "",
      total: 0,
      currentPage: 1,
      pageSize: 9,
      loading: false,
    };
  },
  created() {
    this.teamId = this.$route.query.teamId;
    this.select_Athlete();
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
      this.$http
        .get("gymnastics/athlete/getbyteamid/" + this.teamId)
        .then((res) => {
          this.total = res.data.length;
        });
    },
    //分页查询
    select_Athlete() {
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
          //删除空白行
          this.tableData.splice(0, 1);
          res.data.records.forEach((element) => {
            if (element.teamId == this.teamId) {
              var athlete = {
                id: element.athleteId,
                code: element.athleteCode,
                name: element.athleteName,
                gender: element.athleteGender,
                identifier: element.athleteIdentifier,
                birth: element.athleteBirthyear,
              };
              this.tableData.push(athlete);
            }
          });
          this.loading = false;
        })
        .catch((err) => {
          this.loading = false;
          this.$message.error(err);
        });
    },
    //点击下一页
    handleCurrentChange(val) {
      this.currentPage = val;
      this.select_Athlete();
    },
    //调整size
    handleSizeChange(val) {
      this.pageSize = val;
    },
    //查看运动员信息
    showDetails(index, rows, row) {
      //查看详细信息
      //到home下的EmployeeDetails
      this.$router.push({
        path: "/athleteDetails",
        query: {
          id: row.id,
          team: this.team,
          code: row.code,
          name: row.name,
          gender: row.gender,
          identifier: row.identifier,
          birth: row.birth,
        },
      });
    },
    modifyDetails(index, rows, row) {
      //修改运动员信息
      this.$router.push({
        path: "/athleteModify",
        query: {
          id: row.id,
          team: this.team,
          code: row.code,
          name: row.name,
          gender: row.gender,
          identifier: row.identifier,
          birth: row.birth,
        },
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
          "姓名",
          "性别",
          "身份证号码",
          "出生日期",
        ];
        const filterVal = [
          "id",
          "code",
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
  color: rgb(48, 54, 21);
}
.page {
  display: block;
  position: absolute;
  width: 50%;
  height: 5%;
  right: 5%;
  bottom: 2.5%;
  color: rgb(78, 40, 122);
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