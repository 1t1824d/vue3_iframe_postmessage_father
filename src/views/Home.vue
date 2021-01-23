
<template>
  <div class="Home">
    <div class="Home-content">
      <el-button type="primary" @click="sendMessage" class="buttobox"
        >向 iframe 内发送信息</el-button
      >
      <el-input
        type="textarea"
        :rows="2"
        placeholder="请输入内容"
        v-model="showdata"
      >
      </el-input>
    </div>
    <iframe :src="src" ref="iframe" frameborder="0"></iframe>
  </div>
</template>

<script>
export default {
  data() {
    return {
      src: "https://volodyan.github.io/vue3_iframe_postmessage_son_preview/#/",
      iframeWin: {},
      showdata: "我是父组件数据",
    };
  },
  mounted() {
    this.iframeWin = this.$refs.iframe.contentWindow;
    this.$nextTick(() => {
      // 在外部 Vue 的 window 上添加 postMessage 的监听，并且绑定处理函数 handleMessage
      window.addEventListener("message", this.handleMessage);
    });
  },
  destroyed() {
    // 注意移除监听！注意移除监听！注意移除监听！
    window.removeEventListener("message", this.handleMessage);
  },
  methods: {
    sendMessage() {
      // 外部vue向iframe内部传数据
      this.iframeWin.postMessage(
        {
          cmd: "doSomething"+Math.random()*222,
          params: {
            messge: this.showdata,
            id: "Father",
          },
        },
        "*"
      );
    },
    handleMessage(event) {
      // 根据上面制定的结构来解析 iframe 内部发回来的数据
      const data = event.data;
      console.log("postMessage---iframe 内部发回来的数据", event);
      this.showdata = data.cmd
      switch (data.cmd) {
        case "ready-for-receiving":
          // 业务逻辑
          break;
      }
    },
  },
};
</script>
<style lang="scss" scoped>
.Home {
  display: flex;
  flex-flow: row nowrap;
  justify-content: flex-end;
  align-items: flex-end;
  width: 100%;
  height: 100vh;
  overflow: hidden;
  position: relative;
  .Home-content {
    display: flex;
    flex-flow: column;
    align-items: center;
    position: absolute;
    z-index: 99;
    top: 50%;
    left: 20px;
    .buttobox {
      margin-bottom: 30px;
    }
    .showdatabox {
    }
  }

  iframe {
    margin: 0;
    padding: 0;
    width: calc(100vw - 242px);
    height: calc(100vh - 80px);
    border: 1px solid teal;
    box-sizing: border-box;
    margin-right: 2px;
  }
}
</style>