<template>
  <div class="home">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <div>
      <div id="searchBar">
        <el-button @click="dosearch()" >搜索</el-button>
      </div>
      <Header/>
    </div>
    <div>
      <el-table
        height="500"
        :data="tableData"
        style="width: 100%"
        :row-class-name="tableRowClassName"
      >
        <el-table-column prop="ID" label="ID" ></el-table-column>
        <el-table-column prop="type" label="业务类型"></el-table-column>
        <el-table-column prop="asset" label="币种"></el-table-column>
        <el-table-column prop="user" label="用户名"></el-table-column>
        <el-table-column prop="outAddr" label="外部地址"></el-table-column>
        <el-table-column prop="amount" label="实际金额"></el-table-column>
        <el-table-column prop="fee" label="fee"></el-table-column>
        <el-table-column prop="memo" label="memo"></el-table-column>
        <el-table-column prop="hash" label="hash"></el-table-column>
         <el-table-column prop="cybhash" label="cybhash"></el-table-column>
        <el-table-column prop="status" label="status"></el-table-column>
  <el-table-column
      fixed="right"
      label="操作"
      width="100">
      <template slot-scope="scope">
        <el-button @click="handleClick(scope.row)" type="text" size="small">详情</el-button>
      </template>
    </el-table-column>
      </el-table>
    </div>
    <el-dialog title="搜索" :visible.sync="dialogFormVisible">
            <div >
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="searchConfirm">确 定</el-button>
      </div>
      <el-row>
        <el-col :span="12"><div class="grid-content bg-purple">
           搜索：
           <el-input type="textarea" autosize placeholder="请输入内容" v-model="searchContent"></el-input>
          </div></el-col>
        <el-col :span="12"><div class="grid-content bg-purple-light">
            例子：
           <el-input type="textarea" autosize placeholder="请输入内容" :disabled="true" v-model="preCreateAsset"></el-input>
          </div></el-col>
      </el-row>
    
    </el-dialog>
    <el-dialog title="订单" :visible.sync="updateVisible">
      <el-input type="textarea" autosize placeholder="请输入内容"  v-model="preUpdateAsset"></el-input>
      <div slot="footer" class="dialog-footer">
        <el-button @click="updateVisible = false">好的</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
// @ is an alias to /src
import Header from '@/components/Header.vue'
import axios from "axios";
import searchOrder from "./searchOrder";
let tokenType = "localStorage";
export default {
  name: "home",
  data() {
    return {
      token: "",
      gatewayHost: "http://localhost:8183",
      tableData: [],
      dialogFormVisible: false,
      updateVisible: false,
      searchOrder: searchOrder,
      preCreateAsset: "",
      searchContent: "",
      preUpdateAsset: "",
      search: ""
    };
  },
  components: {
    Header
  },
  methods: {
    handleClick(row){
      let copy = Object.assign({}, row);
      this.preUpdateAsset = JSON.stringify(copy, null, 2);
      this.updateVisible = true;
    },
    tableRowClassName({ row }) {
      if (row.disabled) {
        return "warning-row";
      }
    },
    changeHost(){
      let gatewayHost = prompt(`请输入gatewayHost,当前${this.gatewayHost}`);
      if (gatewayHost != null && gatewayHost !== "null" && gatewayHost !== ""){
        this.gatewayHost = gatewayHost
        this.saveHost(this.gatewayHost)
      }
    },
    isOpenClass(isOpen) {
      if (!isOpen) {
        return "assetClose";
      }
    },
    changeAsset(row) {
      let copy = Object.assign({}, row);
      delete copy["gatewayAccount"];
      delete copy["gatewayPassword"];
      delete copy["withdrawPrefix"];
      delete copy["UpdatedAt"];
      this.preUpdateAsset = JSON.stringify(copy, null, 2);
      this.updateVisible = true;
    },
    cellValueRenderer(row, column, cellValue) {
      let value = cellValue;
      if (typeof row[column.property] === "boolean") {
        value = String(cellValue);
      }
      return value;
    },
    async updateAssetConfirm() {
      let data = null;
      try {
        data = JSON.parse(this.preUpdateAsset);
      } catch (e) {
        alert("不是合法的json");
        return;
      }
      try {
        let s = await axios.post(`${this.gatewayHost}/v1/assets/update`, data, {
          headers: {
            Authorization: "Bearer " + this.token
          }
        });
        console.log(s.data);
        this.updateVisible = false;
        this.loadAssets();
      } catch (e) {
        alert("请求失败");
      }
    },
    async searchConfirm() {
      let data = null;
      try {
        data = JSON.parse(this.searchContent);
      } catch (e) {
        alert("不是合法的json");
        return;
      }
      try {
        this.loadOrders(data);
        this.dialogFormVisible = false;
      } catch (e) {
        alert("请求失败");
      }
    },
    dosearch() {
      this.preCreateAsset = JSON.stringify(searchOrder, null, 2);
      this.searchContent = this.searchContent || JSON.stringify({}, null, 2);
      this.dialogFormVisible = true;
    },
    async loadOrders(query) {
      let s = await axios.post(`${this.gatewayHost}/v1/orders/list`, query, {
        headers: {
          Authorization: "Bearer " + this.token
        }
      });
      console.log(s);
      this.tableData = s.data.data;
    },
    getHost() {
      let gatewayHost = window[tokenType].getItem("gatewayHost");
      // console.log("gatewayHost", gatewayHost);
      if (gatewayHost) {
        this.gatewayHost = gatewayHost;
        return gatewayHost;
      }
      this.gatewayHost = prompt("请输入gatewayHost");
      return this.gatewayHost;
    },
    getToken() {
      let token = window[tokenType].getItem("token");
      // console.log("token", token);
      if (token) {
        this.token = token;
        return token;
      }
      this.token = prompt("请输入token");
      return this.token;
    },
    saveToken(token) {
      window[tokenType].setItem("token", token);
    },
    saveHost(gatewayHost){
    window[tokenType].setItem("gatewayHost", gatewayHost);
    },
    clearToken() {
      window[tokenType].removeItem("token");
      alert("token已经清除");
    }
  },
  async mounted() {
    this.getHost()
    this.getToken()
    this.loadOrders({})
  }
};
</script>
<style scoped>
#searchBar {
  display: inline-block;
  width: 20%;
  margin-left: 10px;
  height: 40px;
}
</style>