/*
 * Copyright 2014-2020 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
    <div class="bg-f7f7f7">
        <!-- <v-content-head :headTitle="$t('title.transactionInfo')" :icon="true" @changGroup="changGroup"></v-content-head> -->
        <div class="module-wrapper">
            <div class="search-part">
                <div class="search-part-left-bg">
                    <!-- <span>{{$t('text.total')}}</span>
                    <span>{{numberFormat(total, 0, ".", ",")}}</span>
                    <span>{{$t('text.tiao')}}</span> -->
                </div>
                <div class="search-part-right">
                    <el-input :placeholder="$t('placeholder.globalSearch')" v-model="searchKey.value"
                        class="input-with-select" clearable @clear="clearText">
                        <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
                    </el-input>
                </div>
            </div>
            <div class="search-table" v-autoTableHeight="160" >
                <el-table height="100%"  :data="transactionList" class="block-table-content" :row-key="getRowKeys"
                    :expand-row-keys="expands" v-loading="loading"  ref="refTable"
                    @expand-change="expandSelect">
                    <el-table-column type="expand" align="center">
                        <template slot-scope="scope">

                            <v-transaction-detail :transHash="scope.row"></v-transaction-detail>
                        </template>
                    </el-table-column>
                    <el-table-column prop="hash" :label="$t('transactionHash')" align="center"
                        :show-overflow-tooltip="true">
                        <template slot-scope="scope">
                            <span>
                                <i class="wbs-icon-copy font-12 copy-key" @click="copyPubilcKey(scope.row['transHash'])"
                                    :title="$t('text.copyHash')"></i>
                                {{ scope.row['hash'] }}
                            </span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="blockNumber" :label="$t('blockHeight')" width="260" align="center"
                        :show-overflow-tooltip="true">
                        <template slot-scope="scope">
                            <!-- 使用过滤器处理数字 -->
                            <span>{{ scope.row['blockNumber'] | blockNumber }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="from" :label="$t('from')" width="280" :show-overflow-tooltip="true"
                        align="center">
                        <template slot-scope="scope">
                            <span>{{ scope.row['from'] }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="to" :label="$t('to')" width="280" :show-overflow-tooltip="true" align="center">
                        <template slot-scope="scope">
                            <span>{{ scope.row['to'] }}</span>
                        </template>
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
import transactionDetail from "@/components/transactionDetail";
import {
    queryHomeSearch,
    BlockNumber, BlockByNumber
} from "@/util/api";
import router from "@/router";
import errcode from "@/util/errcode";
import { numberFormat } from "@/util/util";

export default {
    name: "transaction",
    components: {
        "v-content-head": contentHead,
        "v-transaction-detail": transactionDetail
    },
    data: function () {
        return {
            transactionList: [],
            expands: [],
            searchKey: {
                key: "",
                value: ""
            },
            loading: true,
            numberFormat: numberFormat,
            getRowKeys: function (row) {
                return row.hash;
            },
            txPageId: "1",
            pages: {
                pageSize: 10,
                currentPage: 1,
                total: 0
            },
            txNumber:""
        };
    },


    filters: {
        blockNumber: (value) => {
            return parseInt(value, 16);
        },
    },



    computed: {




    },
    mounted: async function () {
        console.log("liunan")
          if(this.$route.query &&this.$route.query.number){
            console.log(this.$route.query.number.toString())
            this.txNumber = this.$route.query.number.toString();
            var data = {
                jsonrpc: "2.0",
                method: "getBlockByNumber",
                params: [1, this.txNumber, true],
                id: 1,
                txPageId: this.txPageId
            };
        this.getBlockByNumber(data);
          }
    },

    methods: {


        getBlockByNumber(data) {
            console.log(data);
            this.loading = true;
           // 网络请求搜索数据
           BlockByNumber(data).then((res) => {
                this.loading=false
                // TODO 校验搜索框输入逻辑
                this.transactionList = []
                this.transactionList = res.data.result.transactions
                this.pages.total = res.data.totalcount
            },error=>{
                this.loading = false
            })
        },


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
            this.txPageId = val.toString();
            var data = {
                jsonrpc: "2.0",
                method: "getBlockByNumber",
                params: [1, this.txNumber, true],
                id: 1,
                txPageId: this.txPageId
            };
            this.getBlockByNumber(data);
        },

        // 搜索框清除键逻辑
        clearText() {
            this.transactionList = []
        },

        // 搜索按钮逻辑
        search() {
            this.loading=true
            let searchKey = this.searchKey.value
            var arr = Number(searchKey).toString(16);
            var sum = "0x" + arr;
            var data = {
                jsonrpc: "2.0",
                method: "getBlockByNumber",
                params: [1, sum, true],
                id: 1,
                txPageId: "1"
            };

            this.getBlockByNumber(data)
        },

        // 折叠面板每次只能展开一行
        expandSelect(row, expandedRows) {
            var that = this
            if (expandedRows.length) {
                that.expands = []
                if (row) {
                    that.expands.push(row.hash)
                }
            } else {
                that.expands = []
            }
        },


    }
};
</script>
<style scoped>
.block-table-content {
    width: 100%;
    padding-bottom: 16px;
    font-size: 12px;
}

.block-table-content>>>.el-table__expanded-cell {
    padding: 12px 6px;
}

.block-table-content>>>.el-table__expand-icon>.el-icon {
    font-size: 14px;
}

.block-table-content>>>.el-table__row {
    cursor: pointer;
}

.search-part {
    padding: 30px 0px;
    overflow: hidden;
    margin: 0;
}

.search-part::after {
    display: block;
    content: "";
    clear: both;
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
</style>

