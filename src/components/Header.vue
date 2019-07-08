<template>
  <span class="clearToken">
    <el-button @click="changeHost">改变网关host</el-button>
    <el-button @click="clearToken">清除token</el-button>
  </span>
</template>

<script>
let tokenType = "localStorage";
export default {
  name: "HelloWorld",
  data() {
    return {
      token: "",
      gatewayHost: "http://localhost:8183"
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
