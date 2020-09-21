<template>
  <div class="wedding-editor" ref="editor">
    <header class="editor-header">
      <a href="javascript:;"></a>
      <a href="javascript:;" class="minimum"></a>
      <a href="javascript:;" class="maximum"></a>
    </header>
    <!-- 日期 -->
    <p class="code">
      <span style="width: 60%;">
        <b style="font-size:22px !important;color: red;">❤️婚礼邀请函❤️</b>
      </span>
      <span style="width: 40%;">
        <audio
          muted="false"
          style="height: 22px; width: 40%;"
          x5-video-player-type="h5"
          x5-video-player-fullscreen="false"
          controls="controls"
          autoplay="true"
          loop="loop"
          name="media"
        >
          <source src="http://jiemeng.love/Perfect%20Duet.mp3" type="audio/mpeg" />
        </audio>
      </span>
    </p>
    <p class="code">
      <span>{{ startDate }}</span>
    </p>
    <!--代码编辑区-->
    <pre>
      <code v-html="highlightedCode"></code>
    </pre>
    <!-- 打开邀请函 -->
    <div class="editor-open" v-if="(canStart || hasClosed) && !canOpen" @click="canOpen = true">
      <svg
        viewBox="0 0 24 24"
        width="18"
        height="18"
        stroke="currentColor"
        stroke-width="2"
        fill="none"
        stroke-linecap="round"
        stroke-linejoin="round"
        class="css-i6dzq1"
      >
        <rect x="3" y="3" width="18" height="18" rx="2" ry="2" />
        <line x1="9" y1="3" x2="9" y2="21" />
      </svg>
    </div>
    <Executions
      :canExecute="canExecute"
      @onUpdating="scrollToBottom"
      @onFinish="canStart = true;canOpen = true"
    />
    <invitation
      :canOpen="canOpen"
      @onClose="canOpen = false, hasClosed = true"
      @sendBarrage="onAfterSending"
    />
    <Barrage :wish="wish" :canStart="canStart" />
  </div>
</template>

<script>
import Prism from "prismjs";
import "prismjs/themes/prism-okaidia.css";
import "../utils/raf";
import data from "../mock/data";
import wx from "weixin-js-sdk";

import Executions from "./Executions";
import Invitation from "./Invitation";
import Barrage from "./Barrage";
import axios from "axios";

export default {
  name: "Editor",
  components: { Executions, Invitation, Barrage },
  data() {
    
    return {
      startDate: "",
      code: data.code,
      currentCode: "",
      isCursorVisible: 1,
      canExecute: false,
      canOpen: false,
      wish: "",
      hasClosed: false,
      canStart: false
    };
  },
  created() {
    const dateTime = new Date();
    const YY = dateTime.getFullYear();
    const MM =
      dateTime.getMonth() + 1 < 10
        ? "0" + (dateTime.getMonth() + 1)
        : dateTime.getMonth() + 1;
    const D =
      dateTime.getDate() < 10 ? "0" + dateTime.getDate() : dateTime.getDate();
    const hh =
      dateTime.getHours() < 10
        ? "0" + dateTime.getHours()
        : dateTime.getHours();
    const mm =
      dateTime.getMinutes() < 10
        ? "0" + dateTime.getMinutes()
        : dateTime.getMinutes();
    const ss =
      dateTime.getSeconds() < 10
        ? "0" + dateTime.getSeconds()
        : dateTime.getSeconds();
    this.startDate = `${YY}-${MM}-${D} ${hh}:${mm}:${ss}`;
    this.progressivelyTyping();
   
    //分享接口
    // alert("开始获取微信分享授权");
    axios.get("/wechat/getSignature?url=" + location.href.split("#")[0]).then(res => {
      // alert(JSON.stringify(res.data.data));
      // alert("开始配置微信分享接口");
      wx.config({
        debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
        appId: "wx81932768654b29cc", // 必填，公众号的唯一标识
        timestamp: res.data.data.timestamp, // 必填，生成签名的时间戳
        nonceStr: res.data.data.nonceStr, // 必填，生成签名的随机串
        signature: res.data.data.signature, // 必填，签名，见附录1
        jsApiList: ["updateAppMessageShareData", "updateTimelineShareData"] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
      });
    });
    //分享接口
    wx.ready(function() {
      window.console.log("微信接口初始化完成");
      const url = location.href.split("#")[0];
      const title = "朱杰&王梦晴的婚礼邀请函";
      const desc = "10月25日诚挚邀请您来参加我们的婚礼！";
      const imgUrl = "https://www.jiemeng.love/wxshare.jpg";
      //分享接口
      wx.updateAppMessageShareData({
        title: title, // 分享标题
        desc: desc, // 分享描述
        link: url, // 分享链接
        imgUrl: imgUrl, // 分享图标
        success: function() {
          // alert("分享成功！");
        },
        cancel: function() {
          // alert("分享失败！");
        }
      });
      wx.updateTimelineShareData({
        title: title, // 分享标题
        desc: desc, // 分享描述
        link: url, // 分享链接
        imgUrl: imgUrl, // 分享图标
        success: function() {
          // alert("分享成功！");
        },
        cancel: function() {
          // alert("分享失败！");
        }
      });
    });
  },
  updated() {
    this.scrollToBottom();
  },
  computed: {
    highlightedCode() {
      const code = Prism.highlight(
        this.currentCode,
        Prism.languages.javascript
      );
      const codeWithCursor = `${code}<span style="opacity:${this.isCursorVisible}"> ▍</span>`;
      return codeWithCursor;
    }
  },
  methods: {
    scrollToBottom() {
      // 保持页面一直滚到最下面
      this.$refs.editor.scrollTop = 100000;
    },
    // 代码输入
    progressivelyTyping() {
      return new Promise(resolve => {
        let count = 0,
          typingCount = 0,
          typing;
        // 写代码每一帧的函数
        let step = () => {
          let randomNumber = Math.round(Math.random() * 6);
          // 摸你打字的随机速度
          if (count % 2 === 0 && randomNumber % 4 === 0) {
            this.currentCode = this.code.substring(0, typingCount);
            typingCount++;
          }
          // 大约每 24 帧光标闪动一次
          if (count % 24 === 0) {
            this.isCursorVisible = this.isCursorVisible === 0 ? 1 : 0;
          }
          count++;
          if (typingCount <= this.code.length) {
            typing = requestAnimationFrame(step);
          } else {
            resolve();
            this.canExecute = true;
            cancelAnimationFrame(typing);
          }
        };
        typing = requestAnimationFrame(step);
      });
    },
    // 发送弹幕之后
    onAfterSending(wish) {
      this.wish = wish;
      this.canOpen = false;
      setTimeout(() => {
        this.canStart = true;
      }, 800);
    }
    //分享
    /*
    微信分享方法s
    @param{data}:获取的微信加签
    @param{url}:分享的url
    */
    //   wxShare() {

    //   alert("开始获取微信分享授权");
    //   axios.get("/wechat/getSignature").then(res => {
    //     window.console.log(res);
    //     wx.config({
    //       debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    //       appId: "wx81932768654b29cc", // 必填，公众号的唯一标识
    //       timestamp: res.data.data.timestamp, // 必填，生成签名的时间戳
    //       nonceStr: res.data.data.nonceStr, // 必填，生成签名的随机串
    //       signature: res.data.data.signature, // 必填，签名，见附录1
    //       jsApiList: ["updateAppMessageShareData", "updateTimelineShareData"] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
    //     });
    //   });
    //     //分享接口
    //     window.οnlοad = function() {
    //       window.console.log("当前域名是：<%=curUrl%>");
    //     };
    //     wx.ready(function() {
    //       window.console.log("微信接口初始化完成");
    //       const url = "https://www.jiemeng.love";
    //       const title = "朱杰&王梦晴的婚礼邀请函";
    //       const desc = "诚挚邀请您来参加我们的婚礼！";
    //       const imgUrl = "https://www.jiemeng.love/touxiang.png";
    //       //分享接口
    //       wx.updateAppMessageShareData({
    //         title: title, // 分享标题
    //         desc: desc, // 分享描述
    //         link: url, // 分享链接
    //         imgUrl: imgUrl, // 分享图标
    //         success: function() {
    //           alert("分享成功！");
    //         },
    //         cancel: function() {
    //           alert("分享失败！");
    //         }
    //       });
    //       wx.updateTimelineShareData({
    //         title: title, // 分享标题
    //         desc: desc, // 分享描述
    //         link: url, // 分享链接
    //         imgUrl: imgUrl, // 分享图标
    //         success: function() {
    //           alert("分享成功！");
    //         },
    //         cancel: function() {
    //           alert("分享失败！");
    //         }
    //       });
    //     });
    //   }
  }
};
</script>

<style lang="less">
.wedding-editor {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  padding: 15px;
  padding-top: 50px;
  overflow-x: hidden;
  overflow-y: auto;
  z-index: 1;
  transform-origin: 0 0;
  -webkit-transform-origin: 0 0;
  transition: all 1.6s cubic-bezier(0.4, 0, 1, 1);
  -webkit-transition: all 1.6s cubic-bezier(0.4, 0, 1, 1);
  .editor-header {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    padding: 12px;
    overflow: hidden;
    background: #2b2b48;
    z-index: 3;
    > a {
      float: left;
      display: block;
      width: 16px;
      height: 16px;
      margin-right: 5px;
      border-radius: 8px;
      background: #fc615d;
      &.minimum {
        background: #fdbc40;
      }
      &.maximum {
        background: #34c84a;
      }
    }
  }
  p.code {
    margin: 0;
    color: #bbb;
    line-height: 1.2;
    font-family: "Roboto Mono", "Menlo", "Monaco", Courier, monospace !important;
    font-weight: 500 !important;
    font-size: 16px !important;
  }
  pre {
    margin: 0;
    white-space: pre-wrap;
    code {
      white-space: pre-wrap;
      word-break: break-all;
      font-size: 16px !important;
      margin: 0;
      color: #bbb;
      line-height: 1.2;
      font-family: "Roboto Mono", "Menlo", "Monaco", Courier, monospace !important;
      font-weight: 500 !important;
      background: transparent;
    }
  }
  .editor-open {
    position: fixed;
    right: 20px;
    bottom: 20px;
    width: 40px;
    height: 40px;
    padding: 6px;
    border-radius: 20px;
    text-align: center;
    line-height: 18px;
    border: 5px solid #ffd69b;
    color: #a9895d;
    background: #fff1de;
    z-index: 1000;
  }
}
</style>