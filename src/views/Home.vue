<template>
  <div class="home">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <div>
      <el-button @click="addAssetPage">增加</el-button>
      <div id="searchBar">
        <el-input v-model="search" size="mini" placeholder="输入关键字搜索" />
      </div>
       <el-button @click="assetSwitch">开关</el-button>
      <Header/>
    </div>
    <div>
      <el-table
        height="500"
        :data="tableData.filter(data => !search || data.name.toLowerCase().includes(search.toLowerCase()))"
        style="width: 100%"
        @row-click="changeAsset"
        :row-class-name="tableRowClassName"
      >
        <el-table-column prop="ID" label="ID"></el-table-column>
        <el-table-column prop="name" label="币种"></el-table-column>
        <el-table-column :formatter="cellValueRenderer" label="可充值">
          <template slot-scope="scope">
            <span :class="isOpenClass(scope.row.depositSwitch)">{{ scope.row.depositSwitch }}</span>
          </template>
        </el-table-column>
        <el-table-column :formatter="cellValueRenderer" label="可提现">
          <template slot-scope="scope">
            <span :class="isOpenClass(scope.row.withdrawSwitch)">{{ scope.row.withdrawSwitch }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="blockchain" label="区块链"></el-table-column>
        <el-table-column prop="cybname" label="cyb资产"></el-table-column>
        <el-table-column prop="cybid" label="cybid"></el-table-column>
        <el-table-column prop="gatewayAccount" label="网关账号"></el-table-column>
        <el-table-column prop="minDeposit" label="最小充值额"></el-table-column>
        <el-table-column prop="minWithdraw" label="最小提现额"></el-table-column>
        <el-table-column prop="withdrawFee" label="提现手续费"></el-table-column>
      </el-table>
    </div>
    <el-dialog title="创建资产" :visible.sync="dialogFormVisible">
      <el-input type="textarea" autosize placeholder="请输入内容" v-model="preCreateAsset"></el-input>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="createAssetConfirm">确 定</el-button>
      </div>
    </el-dialog>
    <el-dialog title="更新资产" :visible.sync="updateVisible">
      <el-input type="textarea" autosize placeholder="请输入内容" v-model="preUpdateAsset"></el-input>
      <div slot="footer" class="dialog-footer">
        <el-button @click="updateVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateAssetConfirm">确 定</el-button>
      </div>
    </el-dialog>
    <el-dialog title="充提开关" :visible.sync="switchVisible">
      <el-input type="textarea" autosize placeholder="请输入内容" v-model="switchStr"></el-input>
      <div slot="footer" class="dialog-footer">
        <el-button @click="switchVisible = false">取 消</el-button>
        <el-button type="primary" @click="switchConfirm">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import Header from '@/components/Header.vue'
import axios from "axios";
import createAsset from "./createAsset";
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
      createAsset: createAsset,
      preCreateAsset: "",
      preUpdateAsset: "",
      search: "",
      switchVisible:false,
      switchTemplate:{
        deposit:true,
        withdraw:true
      },
      switchStr:""
    };
  },
  components: {
    // HelloWorld
    Header
  },
  methods: {
    tableRowClassName({ row }) {
      if (row.disabled) {
        return "warning-row";
      }
    },
    assetSwitch(){
      this.switchStr = JSON.stringify(this.switchTemplate, null, 2);
      this.switchVisible = true;
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
    async switchConfirm(){
      let data = null;
      try {
        data = JSON.parse(this.switchStr);
      } catch (e) {
        alert("不是合法的json");
        return;
      }
      let s
      try {
         s = await axios.post(`${this.gatewayHost}/v1/assets/switch`, data, {
          headers: {
            Authorization: "Bearer " + this.token
          }
        });
        if (s.data !=="null" && s.data!=null){
          alert(s.data)
        }
        this.switchVisible = false;
        this.loadAssets();
      } catch (e) {
        alert("请求失败"+e);
      }
    },
    async createAssetConfirm() {
      let data = null;
      try {
        data = JSON.parse(this.preCreateAsset);
      } catch (e) {
        alert("不是合法的json");
        return;
      }
      try {
        let s = await axios.post(`${this.gatewayHost}/v1/assets/add`, data, {
          headers: {
            Authorization: "Bearer " + this.token
          }
        });
        console.log(s.data);
        this.dialogFormVisible = false;
        this.loadAssets();
      } catch (e) {
        alert("请求失败");
      }
    },
    addAssetPage() {
      this.preCreateAsset = JSON.stringify(createAsset, null, 2);
      this.dialogFormVisible = true;
    },
    async loadAssets(query) {
      let s = await axios.post(`${this.gatewayHost}/v1/assets/list`, query, {
        headers: {
          Authorization: "Bearer " + this.token
        }
      });
      console.log(s);
      this.tableData = s.data;
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
      window[tokenType].clear("token");
      alert("token已经清除");
    }
  },
  async mounted() {
    this.getHost()
    this.getToken()
    await this.loadAssets({});
  }
};
</script>
<style>
.el-table__row.warning-row {
  background: lightgrey;
}
</style>
<style scoped>
.assetClose {
  color: lightcoral;
}
#searchBar {
  display: inline-block;
  width: 20%;
  margin-left: 10px;
}
</style>