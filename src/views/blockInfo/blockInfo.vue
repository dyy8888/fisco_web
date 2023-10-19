/* * Copyright 2014-2020 the original author or authors. * * Licensed under the
Apache License, Version 2.0 (the "License"); * you may not use this file except
in compliance with the License. * You may obtain a copy of the License at * *
http://www.apache.org/licenses/LICENSE-2.0 * * Unless required by applicable law
or agreed to in writing, software * distributed under the License is distributed
on an "AS IS" BASIS, * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either
express or implied. * See the License for the specific language governing
permissions and * limitations under the License. */
<template>
  <div>
    <!-- <v-content-head
      :headTitle="$t('title.blockTitle')"
      :icon="true"
          @changGroup="changGroup"
        ></v-content-head> -->
    <div class="module-wrapper">
      <div class="search-part">
        <div class="search-part-left-bg">
          <!-- <span>{{ this.$t("text.total") }}</span>
              <span>{{ numberFormat(total, 0, ".", ",") }}</span>
              <span>{{ this.$t("text.tiao") }}</span> -->
        </div>
        <div class="search-part-right">
          <el-input :placeholder="$t('placeholder.globalSearch')" v-model="searchKey.value" class="input-with-select"
            clearable @clear="clearText">
            <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
          </el-input>
        </div>
      </div>
      <div class="search-table" v-autoTableHeight="160">
        <el-table style="width: 100%" height="100%" :data="blockData" class="block-table-content" v-loading="loading"
          ref="refTable" @row-click="link">
          <el-table-column prop="number" :label="$t('BlockHeight')" width="140" align="center">
            <template slot-scope="scope">{{ scope.row && scope.row["number"] | filterBlockHeight }}</template>
          </el-table-column>
          <el-table-column prop="total" :label="$t('TransactionCount')" width="500" align="center">

          </el-table-column>
          <el-table-column prop="sealer" :label="$t('ConsensusNodeHash')" :show-overflow-tooltip="true" align="center">
            <template slot-scope="scope">
              <span>
                <i class="wbs-icon-copy font-12 copy-key" :title="$t('text.copyHash')"></i>
                {{ scope.row | filtersTransactionHash }}
              </span>
            </template>
          </el-table-column>
          <el-table-column prop="timestamp" :label="$t('CreateTime')" width="280" :show-overflow-tooltip="true"
            align="center">
            <template slot-scope="scope">{{ scope.row && scope.row["timestamp"] | filtersTimeStap }}</template>
          </el-table-column>
        </el-table>
        <el-pagination align="center" @size-change="handleSizeChange" @current-change="handleCurrentChange"
          :current-page="pages.currentPage" :page-size="pages.pageSize" layout="total, prev, pager, next, jumper"
          :total="pages.total">
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import contentHead from "@/components/contentHead";
import { GetTransactionNum, BlockByNumber, BlockNumber } from "@/util/api";
import router from "@/router";
import errcode from "@/util/errcode";
import { numberFormat } from "@/util/util";
export default {
  name: "blockInfo",
  components: {
    "v-content-head": contentHead,
  },
  data: function () {
    return {
      blockData: [],
      currentPage: 1,
      pageSize: 10,
      sum: 0,
      total: 0,
      loading: true,
      numberFormat: numberFormat,
      ticket: [],
      conpages: "",
      searchKey: {
        key: "",
        value: "",
      },
      blockPageId: "1",
      pages: {
        pageSize: 10,
        currentPage: 1,
        total: 0
      }
    };
  },
  mounted: function () {
    var data = {
      jsonrpc: "2.0",
      method: "getBlockByNumber_all",
      params: [1, "0x6", true],
      id: 1,
      txPageId: "1",
      blockPageId: this.blockPageId
    };
    this.getList(data);
  },

  filters: {
    filterBlockHeight: (value) => {
      return "Block" + parseInt(value, 16);
    },
    // 格式化transationcount
    filterTransationcount: (value) => {

      console.log(value);
      const data = {
        "jsonrpc": "2.0",
        "method": "getTransactionNum",
        "params": [1, value, true],
        "id": 1
      }
      GetTransactionNum(data).then((res) => {
        console.log(res)
        return res.transactionNum
      }, error => {
        this.loading = false;
      });

    },
    // 格式化copyHash
    filtersTransactionHash: (item) => {
      return item.sealerList[eval(item.sealer).toString(16)];
    },
    // 格式化日期
    filtersTimeStap: (timestamp) => {
      const time = Number(timestamp);
      var date = new Date(time);
      var Y = date.getFullYear() + "-";
      var M =
        (date.getMonth() + 1 < 10
          ? "0" + (date.getMonth() + 1)
          : date.getMonth() + 1) + "-";
      var D =
        (date.getDate() < 10 ? "0" + date.getDate() : date.getDate()) +
        " ";
      var h =
        (date.getHours() < 10 ? "0" + date.getHours() : date.getHours()) +
        ":";
      var m =
        (date.getMinutes() < 10
          ? "0" + date.getMinutes()
          : date.getMinutes()) + ":";
      var s =
        date.getSeconds() < 10
          ? "0" + date.getSeconds()
          : date.getSeconds();
      return Y + M + D + h + m + s;

    }
  },

  methods: {

    async getList(data) {
      try {

        const response = await this.getDataFromBlockByNumber(data);
        const blockNumbers = response.blocks;

        // 遍历数组，向GetTransactionNum接口请求数据，并更新数组的每个元素
        for (let i = 0; i < blockNumbers.length; i++) {
          const blockNumber = blockNumbers[i].number;
          const transactionNumber = await this.getDataFromGetTransactionNum(blockNumber);
          //console.log("transactionNumber" + transactionNumber)

          // 更新res数组的每个元素
          blockNumbers[i].total = transactionNumber
        }
        console.log("页面直接加载或切换getList")
        console.log(blockNumbers)
        this.blockData = blockNumbers;
        this.pages.total = response.blockCount;
        this.loading = false;


      } catch (error) {
        console.log('发生错误：', error);
      }
    },
    getDataFromGetTransactionNum(blockNumber) {
      console.log(blockNumber)
      return new Promise((resolve, reject) => {
        const data = {
          "jsonrpc": "2.0",
          "method": "getTransactionNum",
          "params": [1, blockNumber, true],
          "id": 1
        }
        GetTransactionNum(data).then((res) => {
          resolve(res.data.transactionNum);
        }, error => {
          this.loading = false;
        });
      });
    },


    getDataFromBlockByNumber(data) {
      return new Promise((resolve, reject) => {
        BlockByNumber(data).then((res) => {
          const arr = res.data;

          resolve(arr)

        }, error => {
          this.loading = false;
        });
      });
    },





    async getListForSearch(data) {
      try {
        const response = await this.getDataFromBlockByNumberForSearch(data);
        const blockNumbers = [response.result];

        // 遍历数组，向GetTransactionNum接口请求数据，并更新数组的每个元素
        for (let i = 0; i < blockNumbers.length; i++) {
          const blockNumber = blockNumbers[i].number;
          const transactionNumber = await this.getDataFromGetTransactionNumForSearch(blockNumber);
          console.log("transactionNumber" + transactionNumber)

          // 更新res数组的每个元素
          blockNumbers[i].total = transactionNumber
        }
        this.blockData = [];

        console.log(blockNumbers)
        this.blockData = blockNumbers;
        this.pages.total = 1;//搜索区块时，只有一个
        this.loading = false;


      } catch (error) {
        console.log('发生错误：', error);
      }
    },
    getDataFromGetTransactionNumForSearch(blockNumber) {
      return new Promise((resolve, reject) => {
        const data = {
          "jsonrpc": "2.0",
          "method": "getTransactionNum",
          "params": [1, blockNumber, true],
          "id": 1
        }
        GetTransactionNum(data).then((res) => {


          resolve(res.data.transactionNum);
        }, error => {
          this.loading = false;
        });
      });
    },


    getDataFromBlockByNumberForSearch(data) {
      return new Promise((resolve, reject) => {
        BlockByNumber(data).then((res) => {
          const arr = res.data;

          resolve(arr)

        }, error => {
          this.loading = false;
        });
      });
    },








    // getList(data) {
    //   const seft = this;

    //   this.loading = true;
    //   BlockByNumber(data).then((res) => {



    //     const arr = res.data.blocks;
    //     seft.pages.total = res.data.blockCount

    //     for (let i = 0; i < arr.length; i++) {

    //       const data = {
    //         "jsonrpc": "2.0",
    //         "method": "getTransactionNum",
    //         "params": [1, arr[i].number, true],
    //         "id": 1
    //       }
    //       GetTransactionNum(data).then((res) => {
    //         console.log(res)
    //         arr[i].total = res.data.transactionNum

    //         if (i == arr.length - 1) {
    //           this.loading = false;
    //           seft.blockData = arr;
    //           seft.total = seft.blockData.length;
    //           seft.ticket = seft.blockData;

    //         }
    //       }, error => {
    //         this.loading = false;
    //       });
    //     }


    //   }, error => {
    //     this.loading = false;
    //   });
    // },

    handleData(data) {
      if (!data) {
        return;
      }
    },


    handleSelectionChange(val) {
      this.multipleSelection = val;
    },

    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },

    handleCurrentChange(val) {

      let searchKey = this.searchKey.value
      var data = {}
      if (searchKey == "") {


        this.blockPageId = val.toString();
        data = {
          jsonrpc: "2.0",
          method: "getBlockByNumber_all",
          params: [1, "0x6", true],
          id: 1,
          txPageId: "1",
          blockPageId: this.blockPageId
        };
      } else {
        var arr = Number(searchKey).toString(16);
        var sum = "0x" + arr;
        data = {
          jsonrpc: "2.0",
          method: "getBlockByNumber",
          params: [1, sum, true],
          id: 1,
          txPageId: "1", blockPageId: "1"
        };

      }


      // // 网络请求搜索数据
      // BlockByNumber(data).then((res) => {
      //   this.loading = false;
      //   this.blockData = []
      //   this.blockData = res.data.result.transactions
      //   this.pages.total = res.data.totalcount
      // })

      this.loading = true;

      this.getList(data);
    },

    //跳转到交易详情页
    link: function (row, column, event) {
      this.loading = true;
      router.push({
        path: "/transactionInfo",
        query: {
          number: row.number,
        },
      });
    },


    // 搜索框清除键逻辑
    clearText() {
      this.blockData = []
    },

    // 搜索按钮逻辑
    search() {
      // let searchKey = this.searchKey.value
      // var arr = Number(searchKey).toString(16);
      // var sum = "0x" + arr;
      // var data = {
      //   jsonrpc: "2.0",
      //   method: "getBlockByNumber",
      //   params: [1, sum, true],
      //   id: 1,
      //   txPageId: "1", blockPageId: "1"
      // };
      // this.loading = true;
      // // 网络请求搜索数据
      // BlockByNumber(data).then((res) => {
      //   this.loading = false;
      //   this.blockData = []
      //   this.blockData = res.data.result.transactions
      //   this.pages.total = res.data.totalcount
      // })

      let searchKey = this.searchKey.value
      var data = {}
      if (searchKey == "") {

        data = {
          jsonrpc: "2.0",
          method: "getBlockByNumber_all",
          params: [1, "0x6", true],
          id: 1,
          txPageId: "1",
          blockPageId: this.blockPageId
        };

        this.loading = true;
        this.getList(data);
      } else {
        var arr = Number(searchKey).toString(16);
        var sum = "0x" + arr;
        data = {
          jsonrpc: "2.0",
          method: "getBlockByNumber",
          params: [1, sum, true],
          id: 1,
          txPageId: "1", blockPageId: "1"
        };
        this.loading = true;
        this.getListForSearch(data);
      }


      // // 网络请求搜索数据
      // BlockByNumber(data).then((res) => {
      //   this.loading = false;
      //   this.blockData = []
      //   this.blockData = res.data.result.transactions
      //   this.pages.total = res.data.totalcount
      // })



    },

    // compare(property) {
    //   return function (a, b) {
    //     var value1 = a[property];
    //     var value2 = b[property];
    //     return value2 - value1;
    //   };
    // },
    // // console.log(arr.sort(compare('age')))
    // // 获取当前页的数据信息
    // getPageInfo(item) {
    //   this.currentPage = item; //当前页
    //   var list = (this.currentPage - 1) * this.pageSize; //每去一组数据的第一个索引
    //   this.blockData = this.ticket.slice(list, list + this.pageSize); //从总数据中取出每页的数据
    // },
    // next() {
    //   if (this.currentPage < this.conpages) {
    //     this.currentPage++;
    //     this.getPageInfo(this.currentPage);
    //   }
    // },
    // prav() {
    //   if (this.currentPage != 1) {
    //     this.currentPage--;
    //     this.getPageInfo(this.currentPage);
    //   }
    // },
    // // 区块数量
    // async getBlockNumber() {
    //   var data = {
    //     jsonrpc: "2.0",
    //     method: "getBlockNumber",
    //     params: [1],
    //     id: 1,
    //   };
    //   var info = await BlockNumber(data).then((res) => {
    //     const { data, status } = res;
    //     this.sum = parseInt(data.result, 16);
    //   });

    //   return info;
    // },
    // // changGroup(val) {
    // //   this.getBlockList();
    // // },
    // search: function () {
    //   if (
    //     this.searchKey.key == "pkHash" &&
    //     this.searchKey.value.length != 66 &&
    //     this.searchKey.value
    //   ) {
    //     this.$message({
    //       message: this.$t("text.blockSearchMsg"),
    //       type: "error",
    //       duration: 2000,
    //     });
    //   } else {
    //     this.currentPage = 1;
    //     this.getBlockList();
    //   }
    // },
    // getBlockList: function () {
    //   this.loading = true;
    //   let groupId = localStorage.getItem("groupId");
    //   let reqData = {
    //       groupId: groupId,
    //       pageNumber: this.currentPage,
    //       pageSize: this.pageSize,
    //     },
    //     reqQuery = {};
    //   if (this.$route.query.blockNumber) {
    //     this.searchKey.key = "blockNumber";
    //     this.searchKey.value = this.$route.query.blockNumber;
    //   }
    //   if (this.searchKey.value) {
    //     if (this.searchKey.value.length === 66) {
    //       reqQuery.pkHash = this.searchKey.value;
    //     } else {
    //       reqQuery.blockNumber = this.searchKey.value;
    //     }
    //   }
    // getBlockPage(reqData, reqQuery)
    //   .then((res) => {
    //     this.loading = false;
    //     if (res.data.code === 0) {
    //       this.blockData = res.data.data;
    //       this.total = res.data.totalCount;
    //     } else {
    //       this.$message({
    //         message: this.$chooseLang(res.data.code),
    //         type: "error",
    //         duration: 2000,
    //       });
    //     }
    //   })
    //   .catch((err) => {
    //     this.loading = false;
    //     this.$message({
    //       message: err.data || this.$t("text.systemError"),
    //       type: "error",
    //       duration: 2000,
    //     });
    //   });
  },


  // clickTable: function (row, column, $event) {
  //   console.log(row,'222')
  //   let nodeName = $event.target.nodeName;
  //   if (nodeName === "I") {
  //     return;
  //   }
  //   this.link(row.blockNumber);
  // },
};
</script>
<style scoped>
.search-part {
  padding: 30px 0px;
  overflow: hidden;
  margin: 0;
}

.input-with-select {
  width: 610px;
}

.input-with-select>>>.el-input__inner {
  border-top-left-radius: 20px;
  border-bottom-left-radius: 20px;
  border: 1px solid #eaedf3;
  box-shadow: 0 3px 11px 0 rgba(159, 166, 189, 0.11);
}

.input-with-select>>>.el-input--suffix>.el-input__inner {
  box-shadow: none;
}

.input-with-select>>>.el-input-group__prepend {
  border-left-color: #fff;
}

.input-with-select>>>.el-input-group__append {
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
  box-shadow: 0 3px 11px 0 rgba(159, 166, 189, 0.11);
}

.input-with-select>>>.el-button {
  border: 1px solid #20d4d9;
  border-radius: inherit;
  background: #20d4d9;
  color: #fff;
}

.block-table-content {
  width: 100%;
  padding-bottom: 16px;
  font-size: 12px;
}

.block-table-content>>>.el-table__row {
  cursor: pointer;
}

.nav {
  width: 550px;
  height: 30px;
  background-color: pink;
  display: flex;
  align-items: center;
}

div[class$="Page"] {
  width: 80px;
  height: 25px;
  text-align: center;
  color: black;
  line-height: 25px;
  background-color: coral;
  margin: 0 5px;
}

div[class="pages"] {
  width: 25px;
  height: 25px;
  border: 1px solid #c3c3c3;
  border-radius: 5px;
  margin: 0 5px;
  background-color: rgb(236, 235, 235);
  text-align: center;
  line-height: 25px;
}

.active {
  width: 25px;
  height: 25px;
  border: 1px solid #8f8f8f;
  border-radius: 5px;
  margin: 0 5px;
  background-color: rgb(247, 113, 23);
  text-align: center;
  line-height: 25px;
  color: #fff;
}

.nav div:hover {
  cursor: pointer;
}

.box {
  width: 550px;
  height: 200px;
  border: 1px solid #eee;
  background-color: #f5f5f5;
}
</style>
