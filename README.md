# vue3_ifame_father

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
### 预览页

```
https://volodyan.github.io/vue3_iframe_postmessage_father_preview/#/

```

## postMessage跨域传消息

### 父级框架

```
1.以Home.vue为例，<iframe :src="src" ref="iframe" frameborder="0"></iframe>中的src放入开发环境运行的项目网址或者发布后的网址。

2.内联的框架，就像 <frame> 元素一样，会被包含在 window.frames 伪数组（类数组的对象）中。

3.有了 DOM HTMLIFrameElement 对象，脚本可以通过 contentWindow 访问内联框架的 window 对象。 contentDocument 属性则引用了 <iframe> 内部的 document 元素，(等同于使用contentWindow.document），但IE8-不支持。

4.在框架内部，脚本可以通过 window.parent 引用父窗口对象。

5.脚本访问框架内容必须遵守同源策略，并且无法访问非同源的 window 对象的几乎所有属性。同源策略同样适用于子窗体访问父窗体的 window 对象。跨域通信可以通过 window.postMessage 来实现。

Demo--Home.vue如下：
```

```

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
      src: "http://192.168.1.121:8081/",
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
```


### 子框架

```
1.在子框架main.js中接收父框架通过postMessage传过来的数据，然后将接收的数据保存在vuex中以便使用，同时向父框架传消息，具体代码如下：

```

```
/* *******接收postMessage传值开始******** */

window.addEventListener("message", (e) => {
    if (e.source != window.parent) return;
    let data = {
        cmd: "我是子组件,我接收到了父组件传来的数据，并通知父组件",
        params: {
            messge: "我是子组件",
            id: "son",
        },
    };
    console.log("我是子组件，我终于成功了-------PostMessage", e)
    store.state.postdata = e.data.cmd
    console.log("我store.state.postdata", store.state.postdata)
    window.parent.postMessage(data, "*");
});
/* *******接收postMessage传值结束******** */
```
```
main.js文件


import Vue from 'vue'
import './plugins/axios'
import App from './App.vue'
import router from './router'
import store from './store'
import './plugins/element.js'

/* *******接收postMessage传值开始******** */

window.addEventListener("message", (e) => {
    if (e.source != window.parent) return;
    let data = {
        cmd: "我是子组件,我接收到了父组件传来的数据，并通知父组件",
        params: {
            messge: "我是子组件",
            id: "son",
        },
    };
    console.log("我是子组件，我终于成功了-------PostMessage", e)
    store.state.postdata = e.data.cmd
    console.log("我store.state.postdata", store.state.postdata)
    window.parent.postMessage(data, "*");
});
/* *******接收postMessage传值结束******** */
Vue.config.productionTip = false

new Vue({
    router,
    store,
    render: h => h(App)
}).$mount('#app')

```

```
2.子框架通过postMessage向父框架传消息，通过window.parent.postMessage()方法实现，具体写法如下：
```
```
window.parent.postMessage(
        {
          cmd: this.showdata+Math.random(),
          params: {
            messge: "我是子组件",
            id: "son",
          },
        },
        "*"
      );
```

```
示例代码：



<template>
  <div class="Home">
    <div class="Home_title">我是子组件</div>
    <el-input
      type="textarea"
      :rows="2"
      placeholder="请输入内容"
      v-model="showdata"
      class="textareabox"
    >
    </el-input>
    <div class="postMessage_showdata">{{postMessage_showdata}}</div>
    <el-button type="primary" @click="sendMessage" class="buttobox"
      >向 iframe 外发送信息</el-button
    >
  </div>
</template>

<script>

export default {
  data() {
    return {
      showdata: "我是子组件传来的数据",
     
    };
  },
computed:{
  postMessage_showdata(){
    return this.$store.state.postdata
  }
},
  methods: {
    sendMessage() {
      window.parent.postMessage(
        {
          cmd: this.showdata+Math.random(),
          params: {
            messge: "我是子组件",
            id: "son",
          },
        },
        "*"
      );
    },
  },
};
</script>
<style lang="scss" scoped>
.Home {
  display: flex;
  flex-flow: column;
  align-items: center;

  width: 100%;
  .Home_title {
    width: 10%;
    margin-top: 160px;
    font-size: 24px;
    font-weight: bold;
    color: teal;
  }
  .postMessage_showdata{
    border: 1px solid teal;
     width: 20%;
     height: 60px;
     overflow: auto;
      margin-top: 60px;
  }
  .textareabox {
    width: 20%;
    margin-top: 60px;
  }
  .buttobox {
    margin-top: 60px;
    width: 10%;
  }
}
</style>

```
