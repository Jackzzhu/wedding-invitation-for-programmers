<template>
  <div class="wedding-invitation" :class="{ 'invitation-bounce':canOpen }">
    <div class="invitation-container" :class="{ 'invitation-down':isOpening }">
      <div class="invitation-cover" id="content">
        <div class="cover-content" :class="{'invitation-up':isOpening}">
          <div class="content-inside" >
            <viewer :images="photo">
              <img v-for="(src,index) in photo" :src="src" :key="index" :onerror="errorImg" />
            </viewer>
            <p>
              <b>朱杰 & 王梦晴</b>，我们结婚啦！
            </p>
            <p>
              <b>诚挚邀请您参加我们的婚礼！</b>
            </p>
            <p>
              时间：
              <b>2020年10月25日 中午11:58</b>
            </p>
            <p>
              地点：
              <b>睢宁县香格里拉大酒店</b> &#12288;
              <a :href="'http://m.amap.com/search/mapview/keywords=睢宁县香格里拉大酒店'">导航</a>
            </p>
            <p>
              <b>
                <a :href="'tel:15351685016'">联系新郎</a>&#12288;
                <a :href="'tel:13775829779'">联系新娘</a>
              </b>
            </p>
            <div class="content-inside-bless">
              <input
                placeholder="写下你的祝福"
                @keyup.enter="sendBarrage"
                @focus="isFocused = true"
                @blur="isFocused = false, hasEntered = false"
                v-model="wish"
                ref="wishInput"
              />
              <p v-if="!wish && isFocused && hasEntered">请输入祝福哦</p>
              <div>
                <button @click="sendBarrage">发送祝福弹幕</button>
                <button @click="closeInvitation">关闭</button>
              </div>
            </div>
          </div>
        </div>
        <div class="cover-inside-left" :class="{'opening':isOpening}">
          <img style="position: relative;height:20%;top:25%;left:15%;s" src="../images/心心.png" />
        </div>
        <div class="cover-inside-right" :class="{'opening':isOpening}"></div>
        <img
          class="cover-inside-seal"
          src="../images/seal.png"
          @click="openInvitation"
          :class="{'invitation-flight':isOpening}"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import Viewer from "v-viewer";
import "viewerjs/dist/viewer.css";
import axios from "axios";

Vue.use(Viewer);
Viewer.setDefaults({
  Options: {
    inline: true,
    button: true,
    navbar: true,
    title: true,
    toolbar: true,
    tooltip: true,
    movable: true,
    zoomable: true,
    rotatable: true,
    scalable: true,
    transition: true,
    fullscreen: true,
    keyboard: true,
    url: "data-source"
  }
});

export default {
  props: ["canOpen"],
  data() {
    return {
      isOpening: false,
      wish: "",
      isFocused: false,
      hasEntered: false,
      photo: [
        "https://www.jiemeng.love/aixin.png",
        "https://www.jiemeng.love/jack.png",
        "https://www.jiemeng.love/huabian.png",
        "https://www.jiemeng.love/touxiang.png",
        "https://www.jiemeng.love/huabian.png",
        "https://www.jiemeng.love/zhongshi.jpeg",
        "https://www.jiemeng.love/huabian.png",
        "https://www.jiemeng.love/qipao.jpeg",
        "https://www.jiemeng.love/huabian.png",
        "https://www.jiemeng.love/hunsha.jpeg"
      ],

      errorImg: "https://www.jiemeng.love/touxiang.png"
    };
  },
  methods: {
    // 打开邀请函
    openInvitation() {
      this.isOpening = true;
    },
    closeInvitation() {
      this.isOpening = false;
      setTimeout(() => {
        this.$emit("onClose");
      }, 660);
    },
    // 发送弹幕
    sendBarrage() {
      this.$nextTick(() => {
        axios.get("/greet/add?" + this.wish);
        this.hasEntered = true;
        if (!this.wish) {
          return;
        }
        this.isOpening = false;
        this.$refs.wishInput.blur();
        setTimeout(() => {
          this.$emit("sendBarrage", this.wish);
        }, 660);
      });
    },
  }
};
</script>

<style lang="less">
.wedding-invitation {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  padding: 30px 20px;
  padding-top: 60px;
  z-index: 4;
  transform: scale(0.05);
  -webkit-transform: scale(0.05);
  opacity: 0;
  transition: transform 0.8s cubic-bezier(0.26, 1.84, 0.39, 0.61),
    opacity 0.5s linear;
  -webkit-transition: -webkit-transform 0.8s
      cubic-bezier(0.26, 1.84, 0.39, 0.61),
    opacity 0.5s linear;
  background-size: 100%;
  overflow: hidden;
  &.invitation-bounce {
    opacity: 1;
    transform: scale(1);
    -webkit-transform: scale(1);
  }
  .invitation-container {
    position: relative;
    width: 100%;
    height: 100%;
    transition: transform 0.6s cubic-bezier(0.4, 0, 1, 1);
    -webkit-transition: -webkit-transform 0.6s cubic-bezier(0.4, 0, 1, 1);
    &.invitation-down {
      transform: translateY(20px);
      -webkit-transform: translateY(20px);
    }
    .invitation-cover {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: #d65047;
      border-radius: 10px;
      perspective: 500px;
      box-shadow: 0 0 20px 2px rgba(0, 0, 0, 0.15);
      .cover-content {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        padding: 10px 20px;
        transition: transform 0.6s cubic-bezier(0.4, 0, 1, 1);
        -webkit-transition: -webkit-transform 0.6s cubic-bezier(0.4, 0, 1, 1);
        &.invitation-up {
          transform: translateY(-60px);
          -webkit-transform: translateY(-60px);
        }
        .content-inside {
          height: 100%;
          padding: 20px;
          color: #a9895d;
          background-color: #fff1de;
          text-align: center;
          overflow: auto;
          .content-inside-photo {
            width: 100%;
            margin-bottom: 10px;
            padding: 5px;
            border: 1px solid #f7debb;
          }
          p {
            margin-top: 0;
            margin-bottom: 5px;
          }
          .content-inside-bless {
            input {
              width: 100%;
              height: 35px;
              margin-bottom: 10px;
              outline: none;
              border: none;
              border-bottom: 1px solid #f7debb;
              color: #a9895d;
              background: transparent;
              &::-webkit-input-placeholder {
                color: #e8d1b1;
                font-size: 12px;
              }
              &::-moz-placeholder {
                color: #e8d1b1;
                font-size: 12px;
              }
              &:-ms-input-placeholder {
                color: #e8d1b1;
                font-size: 12px;
              }
              &:-moz-placeholder {
                color: #e8d1b1;
                font-size: 12px;
              }
            }
            > div {
              display: flex;
              button {
                width: 100%;
                height: 35px;
                color: #a9895d;
                background: #f7debb;
                border: none;
                outline: none;
                &:disabled {
                  opacity: 0.8;
                }
                &:first-child {
                  margin-right: 10px;
                  flex: 1;
                }
                &:last-child {
                  width: 60px;
                  border: 1px solid #f7debb;
                  background: transparent;
                }
              }
            }
          }
        }
      }
      .cover-inside-left {
        position: absolute;
        left: 0;
        top: 0;
        width: 70%;
        height: 100%;
        border-radius: 10px;
        background-color: #d65047;
        box-shadow: 5px 0 10px rgba(0, 0, 0, 0.2);
        z-index: 6;
        transition: transform 0.5s;
        -webkit-transition: -webkit-transform 0.5s;
        transform-origin: 0 50%;
        -webkit-transform-origin: 0 50%;
        &.opening {
          transform: rotate3d(0, 1, 0, -140deg);
          -webkit-transform: rotate3d(0, 1, 0, -140deg);
        }
      }
      .cover-inside-right {
        position: absolute;
        right: 0;
        top: 0;
        width: 40%;
        height: 100%;
        border-radius: 10px;
        background-color: #d65047;
        box-shadow: -5px 0 10px rgba(0, 0, 0, 0.2);
        z-index: 5;
        transition: transform 0.5s;
        -webkit-transition: -webkit-transform 0.5s;
        transform-origin: 100% 50%;
        -webkit-transform-origin: 100% 50%;
        &.opening {
          transform: rotate3d(0, 1, 0, 140deg);
          -webkit-transform: rotate3d(0, 1, 0, 140deg);
        }
      }
      .cover-inside-seal {
        position: absolute;
        left: 70%;
        bottom: 100px;
        width: 80px;
        height: 80px;
        margin-left: -40px;
        z-index: 7;
        transform-origin: 50% 50%;
        -webkit-transform-origin: 50% 50%;
        transition: all 0.8s cubic-bezier(0.4, 0, 1, 1);
        -webkit-transition: all 0.8s cubic-bezier(0.4, 0, 1, 1);
        &.invitation-flight {
          opacity: 0;
        }
      }
    }
  }
}
</style>
