<template>
  <div>
    <!-- 队医表格 -->
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
        prop="doctorID"
        label="队医ID"
        width="200"
        align="center"
      ></el-table-column>

      <el-table-column prop="name" label="姓名" width="300" align="center">
      </el-table-column>

      <el-table-column
        prop="identifier"
        label="身份证号码"
        width="300"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="phone"
        label="电话号码"
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
          doctorID: "",
          name: "",
          identifier: "",
          phone: "",
        },
      ],
      total: 0,
      currentPage: 1,
      pageSize: 9,
      loading: false,
    };
  },
  created() {
    this.select_user();
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
    //获取队医总数
    totalData() {
      this.$http.get("gymnastics/doctor/findAll").then((res) => {
        this.total = res.data;
      });
    },
    //分页查询
    select_user() {
      this.totalData();
      this.loading = true;
      this.$http
        .get(
          "gymnastics/doctor/getbypage/" +
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
              doctorID: element.doctorId,
              name: element.doctorName,
              identifier: element.doctorIdentifier,
              phone: element.doctorPhone,
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
    //导出队医excel
    update() {
      this.export2Excel();
    },
    //导出excel方法
    export2Excel() {
      require.ensure([], () => {
        const { export_json_to_excel } = require("../../excel/Export2Excel.js");
        //设置Excel的表格第一行的标题
        const tHeader = ["队医ID", "姓名", "身份证号码", "电话号码"];
        const filterVal = ["doctorID", "name", "identifier", "phone"];
        const list = this.tableData; //把data里的tableData存到list
        const data = this.formatJson(filterVal, list);
        export_json_to_excel(tHeader, data, "队医表");
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