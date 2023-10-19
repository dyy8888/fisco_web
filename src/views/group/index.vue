
<template>
  <div>
    <div class="module-wrapper">
      <div class="search-table" v-autoTableHeight="150">
        <el-table max-height="100%"
          :data="nodeData"
          class="search-table-content"
          v-loading="loading"
        >
          <el-table-column prop="nodeId" label="NodeID" show-overflow-tooltip width="400">
          </el-table-column>
          <el-table-column prop="type" label="NodeType"> </el-table-column>
          <el-table-column prop="kg" label="BlockHeight"> </el-table-column>
          <el-table-column prop="view" label="pbftView"> </el-table-column>
          <el-table-column prop="stutas" label="WorkState"> </el-table-column>

          <!-- <el-table-column label="操作">
            <template slot-scope="scope">
              <span
                style="margin-right: 10px; color: #2fcdd1"
                @click="handleDelete(scope.$index, scope.row)"
                >修改</span
              >
              <span
                style="color: #2fcdd1"
                @click="handleDelete(scope.$index, scope.row)"
                >备注</span
              >
            </template>
          </el-table-column> -->
        </el-table>
        <div style="height: 20px"></div>
      </div>
    </div>
  </div>
</template>

<script>
import contentHead from "@/components/contentHead";
import { NodeList } from "@/util/api";
import Bus from "@/bus";
export default {
  name: "group",
  components: {
    "v-content-head": contentHead,
  },
  data() {
    return {
      nodeData: [

      ],
      loading: false,
      group: localStorage.getItem("groupId") || null,
    };
  },

  mounted() {
    this.getNodeTable();
  },
  methods: {


    getNodeTable() {
      var data = {
        jsonrpc: "2.0",
        method: "getConsensusStatus",
        params: [1],
        id: 1,
      };
      NodeList(data).then((res) => {

        const { data } = res;
        const kg = data.result[0].highestblockNumber
        this.nodeData = data.result[1].map(it=>({
          ...it,
          type:"ConsensusNode",
          stutas:"Normal",
          kg:kg
        }));



      });

    },
    textColor(val) {
      let colorString = "";
      switch (val) {
        case 1:
          colorString = "#58cb7d";
          break;
        case 2:
          colorString = "#ed5454";
          break;
      }
      return colorString;
    },
    nodesStatus(val) {
      let transString = "";
      switch (val) {
        case 1:
          transString = this.$t("table.run");
          break;
        case 2:
          transString = this.$t("table.abnormal");
          break;
      }
      return transString;
    },
    copyNodeIdKey(val) {
      if (!val) {
        this.$message({
          type: "fail",
          showClose: true,
          message: this.$t("notice.copyFailure"),
          duration: 2000,
        });
      } else {
        this.$copyText(val).then((e) => {
          this.$message({
            type: "success",
            showClose: true,
            message: this.$t("notice.copySuccessfully"),
            duration: 2000,
          });
        });
      }
    },
  },
};
</script>
<style scoped>
.module-wrapper {
  padding: 30px;
  height: 100%;
  background-color: #20293c;
}
.search-part {
  padding: 30px 41px 18px 42px;
  overflow: hidden;
}
.search-part::after {
  display: block;
  content: "";
  clear: both;
}
.search-part-left {
  float: left;
}
.search-part-left-btn {
  border-radius: 20px;
}
.search-part-right {
  float: right;
}
.search-table {
  /* padding: 0 40px 0 41px; */
  background: #fff;
  font-size: 15px;
  -webkit-box-shadow: 0 4px 12px 0 #dfe2e9;
  box-shadow: 0 4px 12px 0 #dfe2e9;

  border-radius: 4px;
}
.search-table-content {
  width: 100%;
  padding: 20px;
  border-radius: 4px;

  /* background-color: #20293c; */
}
.search-table-content::before {
  width: 0;
}
/* .el-table::before { */
.search-table-content >>> td,
.search-table-content >>> th {
  padding: 8px 0;
  font-size: 12px;
}
.search-table-content >>> th {
  color: #000;
}
.search-table-content >>> td {
  color: #737a86;
}
.search-table-detail {
  width: 91%;
  float: right;
}
.search-table-detail >>> td,
.search-table-detail >>> th {
  color: #737a86;
}
.input-with-select >>> .el-input__inner {
  border-top-left-radius: 20px;
  border-bottom-left-radius: 20px;
  border: 1px solid #eaedf3;
  box-shadow: 0 3px 11px 0 rgba(159, 166, 189, 0.11);
}
.input-with-select >>> .el-input-group__append {
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
  box-shadow: 0 3px 11px 0 rgba(159, 166, 189, 0.11);
}
.input-with-select >>> .el-button {
  border: 1px solid #1f83e7;
  border-radius: inherit;
  background: #1f83e7;
  color: #fff;
}
</style>
