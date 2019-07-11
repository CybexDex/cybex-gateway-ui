<template>
  <span class="clearToken">
    <el-button @click="changeHost">改变网关host</el-button>
    <el-button @click="clearToken">清除token</el-button>
     <el-button @click="saveEnv">保存环境</el-button>
       <el-select v-model="envName" placeholder="请选择"  @change="useEnv">
    <el-option
      v-for="(index,item) in envs"
      :key="item"
      :label="item"
      :value="item">
    </el-option>
  </el-select>
  </span>
</template>

<script>
let tokenType = "localStorage";
export default {
  name: "HelloWorld",
  data() {
    return {
      token: "",
      gatewayHost: "http://localhost:8183",
      envName: "",
      envs:{}
    };
  },
  methods: {
    tableRowClassName({ row }) {
      if (row.disabled) {
        return "warning-row";
      }
    },
    changeHost() {
      let gatewayHost = prompt(`请输入gatewayHost,当前${this.gatewayHost}`);
      if (gatewayHost != null && gatewayHost !== "null" && gatewayHost !== "") {
        this.gatewayHost = gatewayHost;
        this.saveHost(this.gatewayHost);
      }
    },
    loadEnv(){
      this.envName = window[tokenType].getItem("envName") || ''
      let s = window[tokenType].getItem("envs") || '{}'
      this.envs = JSON.parse(s)
      return this.envs
    },
    useEnv(){
      this.gatewayHost = this.envs[this.envName].gatewayHost
      this.token = this.envs[this.envName].token
      window[tokenType].setItem("envName",this.envName)
      this.saveToken(this.token)
      this.saveHost(this.gatewayHost)
      window.location.reload();
    },
    saveEnv(){
      let input = prompt(`请输入环境名,当前${this.envName}`);
        if (input != null && input !== "null" && input !== "") {
          this.envName = input
          let obj = {
             token: this.token,
             gatewayHost: this.gatewayHost,
          }
          this.envs = this.loadEnv() || {}
          this.envs[input] = obj
          window[tokenType].setItem("envs",JSON.stringify(this.envs))
        }
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
    saveHost(gatewayHost) {
      window[tokenType].setItem("gatewayHost", gatewayHost);
    },
    clearToken() {
      window[tokenType].removeItem("token");
      alert("token已经清除");
    }
  },
  async mounted() {
    let gatewayHost = this.getHost();
    let token = this.getToken();
    this.loadEnv();
    if (gatewayHost != null && gatewayHost !== "null" && gatewayHost !== "") {
      if (token != null && token !== "null" && token !== "") {
        this.saveToken(token);
        this.saveHost(gatewayHost);
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.clearToken {
  position: absolute;
  right: 10px;
}
</style>
