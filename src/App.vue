<template>
  <div id="app">
    <!--    <nav>-->
    <!--      <router-link to="/">Home</router-link> |-->
    <!--      <router-link to="/about">About</router-link>-->
    <!--    </nav>-->
    <router-view v-if="isRouterAlive" />
  </div>
</template>

<script>
import { log } from '@antv/g2plot/lib/utils';
import { getRequest } from './api/user';
import { mapActions } from 'vuex';
export default {
  data() {
    return {
      isRouterAlive: true
    }
  },

  created() {
    this.init();
  },

  mounted() {

  },

  methods: {
    ...mapActions(["getDataList", "getTaskList", "getModelList"]),
    init() {
      //获取所有数据表信息
      this.getDataList();
      //因为用户登录后只展示登录用户所创建的任务，所以getTaskList在登录后跳转组件时调用
      //this.getTaskList();
      this.getModelList();
      console.log("search="+location.search);
      let url = location.search;
      let urlParam = new URLSearchParams(url);
      let repkeyValue = urlParam.get("repKey");
      console.log("截取后="+ repkeyValue);
      getRequest(`/login?repKey=${repkeyValue}`).then((resp) => {
        if (resp) {
          console.log("后台回复的code" + resp.code);
          console.log("后台回复的UserName" + resp.data.respBody.UserName);
          console.log("后台回复的UserCode" + resp.data.respBody.UserCode);
          if (resp.code == "200") {
            sessionStorage.setItem("username", resp.data.respBody.UserName);
            sessionStorage.setItem("userid", resp.data.respBody.UserCode);
            this.$router.push("/sideBar/SoftwareIntro");
          }
        } else {
          this.$message.error("用户不存在或者密码错误");
        }
      });

    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* //text-align: center; */
  color: #2c3e50;
}

* {
  margin: 0;
  padding: 0;
  border: 0
}

html,
body {
  height: 100%;
}

/* //注明的样式 */
.explain {
  color: red;
  font-size: 12px;
}

/* //表单头的样式 */
.whiteItem .el-form-item__label {
  color: #fff;
}

.title {
  font-size: 26px;
  margin: 10px auto 10px auto;
  text-align: center;
  font-weight: bold;
}

.caret-wrapper {
  padding: 0px;
}

.InfoData {
  width: 100%;
  margin: 10px auto;
  padding: 0;
}

.el-table {
  ::v-deep th {
    padding: 0;
  }

  ::v-deep td {
    padding: 0;
  }
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}

/* //::-webkit-scrollbar {
//  width: 0 !important;
//}
//::-webkit-scrollbar {
//  width: 0 !important;
//  height: 0;
//}
//html,body,#app {
//  height: 100%;
//} */</style>

