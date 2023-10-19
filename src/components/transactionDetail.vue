<template>
  <div>
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane :label="$t('table.transactionInfo')" name="txInfo">
        <el-row v-for="item in txInfoList" :key="item">
          <el-col :xs="24" :sm="24" :md="6" :lg="12">
            <span>{{ item }}：{{ transHash[item] }}</span>
          </el-col>
          <!-- <el-col :xs='24' :sm="24" :md="18" :lg="20" :xl="22">

                        <template v-if="item=='input'">
                            <el-input v-show="showDecode" type="textarea" :autosize="autosizeMao" v-model="txInfoMap[item]">
                            </el-input>
                            <div v-show="!showDecode" class="input-data">
                                <div class="input-label">
                                    <span class="label">function</span>
                                </div>
                                <div class="input-label">
                                    <span class="label">methodId</span>
                                </div>
                                <div class="input-label">
                                    <span class="label">data</span>
                                </div>
                            </div>
                        </template>
                        <template v-else-if="item=='to'">
                            <p class="base-p">{{txInfoMap[item]}} <span v-if="txInfoMap[item]">{{txInfoMap[item] | contractSource}}</span></p>
                        </template>
                        <template v-else>
                            <p class="base-p">{{txInfoMap[item]}}</p>
                        </template>
                    </el-col> -->
        </el-row>
        <div class="input-conter">
          <span>input: </span>
          <div class="input-fix">
            <el-row>
              <el-col
                :xs="24"
                :sm="24"
                :md="24"
                :lg="24"
                style="margin-bottom: 10px"
              >
                <span>function</span>
                <span style="margin-left: 57px">{{ info.function }}</span>
              </el-col>
              <el-col
                :xs="24"
                :sm="24"
                :md="24"
                :lg="24"
                style="margin-bottom: 10px"
              >
                <span>methodId</span>
                <span style="margin-left: 50px"> {{ info.methodId}}</span>
              </el-col>
            </el-row>
            <el-col :xs="24" :sm="24" :md="24" :lg="24" style="display: flex">
              <span>data</span>
              <div style="width: 100%; margin: -12px 0 0 50px">
<<<<<<< HEAD
<<<<<<< HEAD
                <el-table :data="tableData" 
                style="width: 90% ;" 
                max-height="150"
                >
=======
                <el-table :data="tableData" style="width: 100%" >
>>>>>>> parent of eb8b091 (1、修改英文table为中文)
=======
                <el-table :data="tableData" style="width: 100%" >
>>>>>>> parent of eb8b091 (1、修改英文table为中文)

                  <el-table-column
                    v-for="(item,index) in datakey" :key="index"
                    :prop="item"
                    :label="item"
                    width="180"
                  >

                  </el-table-column>
                </el-table>
              </div>
            </el-col>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane :label="$t('table.transactionReceipt')" name="txReceiptInfo">
        <el-row v-for="item in txReceiptInfoList" :key="item">
          <el-col :xs="24" :sm="24" :md="6" :lg="12">
            <span>{{ item }}：{{ transHash[item] }}</span>
          </el-col>
          <el-col :xs="24" :sm="24" :md="18" :lg="20" :xl="22">
            <template v-if="item == 'logs'">
              <p class="base-p" v-html="txInfoReceiptMap[item]"></p>
              <div v-show="!showReceiptDecode">解码后</div>
            </template>
            <template v-else-if="item == 'status'">
              <p
                class="base-p"
                :style="{ color: txStatusColor(txInfoReceiptMap[item]) }"
              >
                {{ txInfoReceiptMap[item] }}
              </p>
            </template>
            <template v-else-if="item == 'to'">
              <p class="base-p">
                {{ txInfoReceiptMap[item] }}
                <span v-if="txInfoReceiptMap[item]">{{
                  txInfoReceiptMap[item] | contractSource
                }}</span>
              </p>
            </template>
            <template v-else>
              <p class="base-p">{{ txInfoReceiptMap[item] }}</p>
            </template>
          </el-col>
        </el-row>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
<script>
import { queryTxInfo, queryTxReceiptInfo, txInfo } from "@/util/api";
export default {
  name: "transactionDetail",
  props: {
    transHash: {
      type: String,
    },
  },
  data() {
    return {
      activeName: "txInfo",
      showDecode: true,
      showReceiptDecode: true,
      btnText: "解码",
      btnReceiptText: "解码",
      autosizeMao: {
        minRows: 2,
        maxRows: 5,
      },
      txInfoMap: {},
      txInfoReceiptMap: {},
      group: localStorage.getItem("groupId")
        ? localStorage.getItem("groupId")
        : "1",
      txInfoList: [
        "blockHash",
        "blockNumber",
        "gas",
        "from",
        "transactionIndex",
        "to",
        "nonceRaw",
        "value",
        "hash",
        "gasPrice",
        // "input",
      ],
      txReceiptInfoList: [
        "output",
        "blockHash",
        "gasUsed",
        "blockNumber",
        "contractAddress",
        "from",
        "transactionIndex",
        "to",
        "logsBloom",
        "transactionHash",
        "status",
        "logs",
      ],
      tableData: [

      ],
      info:{
        function:"",
        methodId:""
      },
      dataList:[],
      datakey:{

      }
    };
  },
  mounted() {
    // this.getTxInfo();
    this.txInfo();
  },
  methods: {
<<<<<<< HEAD
<<<<<<< HEAD
    /*
    table 双击复制
    */
    // copyText(row, column, cell, event) {
    //   // 双击复制
    //   let save = function (e) {
    //     e.clipboardData.setData('text/plain', event.target.innerText);
    //     e.preventDefault();  //阻止默认行为
    //   }
    //   document.addEventListener('copy', save);//添加一个copy事件
    //   document.execCommand("copy");//执行copy方法
    //   this.$message({ message: '复制成功', type: 'success' })//提示
    // },
=======
>>>>>>> parent of eb8b091 (1、修改英文table为中文)
=======
>>>>>>> parent of eb8b091 (1、修改英文table为中文)
    txInfo() {

      const arr = '0x' + this.transHash.blockNumber.toString(16)
      var data = {
        jsonrpc: "2.0",
        method: "getBlockByNumber_sp",
        params: [1, arr, true],
        id: 1,
      };
      txInfo(data).then((res) => {
        this.info.methodId = res.data.transactions[0].method
        this.dataList = res.data.transactions
        this.datakey = Object.keys(this.dataList[0])
        const arr = this.dataList.filter(map=>this.transHash.hash===map.txHash)
        this.tableData = arr;
      });
    },
    // getTxInfo() {
    //   console.log(this.transHash, "transHash");
    //   // console.log(this.txInfoList,'222')
    //   // this.txInfoList = this.transHash
    //   // queryTxInfo(this.group, this.transHash)
    //   //     .then(res => {
    //   //         const { data, status } = res;
    //   //         if (status === 200) {
    //   //             this.txInfoMap = data;
    //   //         } else {
    //   //             this.$message({
    //   //                 type: "error",
    //   //                 message: this.$chooseLang(res.data.code)
    //   //             });
    //   //         }
    //   //     })
    //   //     .catch(err => {
    //   //         this.$message({
    //   //             type: "error",
    //   //             message: err.data || this.$t('text.systemError')
    //   //         });
    //   //     });
    // },
    deCodeInput() {
      this.showDecode = !this.showDecode;
      if (!this.showDecode) {
        this.btnText = "还原";
      } else {
        this.btnText = "解码";
      }
    },
    // deCodeLogs() {
    //     this.showReceiptDecode = !this.showReceiptDecode;
    //     if (!this.showReceiptDecode) {
    //         this.btnReceiptText = "还原";
    //     } else {
    //         this.btnReceiptText = "解码";
    //     }
    // },
    decodeInputfun() {},
    decodeLogsFun() {},
    handleClick(tab) {
      if (tab.name == "txReceiptInfo") {
        // this.getTxReceiptInfo()
      }
    },
    // getTxReceiptInfo() {
    //     queryTxReceiptInfo(this.group, this.transHash)
    //         .then(res => {
    //             const { data, status } = res;
    //             if (status === 200) {
    //                 this.txInfoReceiptMap = data;
    //             } else {
    //                 this.$message({
    //                     type: "error",
    //                     message: this.$chooseLang(res.data.code)
    //                 });
    //             }
    //         })
    //         .catch(err => {
    //             this.$message({
    //                 type: "error",
    //                 message: err.data || this.$t('text.systemError')
    //             });
    //         });
    // },
    txStatusColor(val) {
      if (val == "0x0") {
        return "#67C23A";
      } else {
        return "#F56C6C";
      }
    },
  },
};
</script>
<style scoped>
.base-p {
  overflow: hidden;
  word-break: break-all;
  word-wrap: break-word;
}
.input-conter {
  display: flex;
}
.input-fix {
  width: 90%;
  margin: 5px 0 0 15px;
  /* margin-left: 15px; */
  border: 1px solid #ddd;
  padding: 15px 20px;
}
</style>
