<!--
 * @Author: BabyChin
 * @Date: 2020-09-20 22:00:45
 * @LastEditTime: 2020-10-07 13:45:22
 * @Description: Clock
-->
<template>
  <div :class="['clock-app', { tomato: mode === 2 }, { light: skin === 2 }]">
    <div class="tools-bar">
      <i
        :class="['ico', mode === 1 ? 'ico-tomato' : 'ico-clock']"
        @click="operate('tomato')"
        >&nbsp;{{ mode === 1 ? "番茄钟" : "时钟" }}
      </i>
      <i
        :class="['ico', skin === 1 ? 'ico-sun' : 'ico-moon']"
        @click="operate('skin')"
        v-if="mode === 1"
        >&nbsp;{{ skin === 1 ? "浅色" : "暗黑" }}</i
      >
    </div>
    <!-- 时钟 -->
    <template v-if="mode === 1">
      <h3 class="clock-hours">
        <span class="hour">{{ clock.hour }}</span>
        <em :class="{ breath: clock.breath }">:</em>
        <span class="minute">{{ clock.minute }}</span>
      </h3>
      <p class="clock-dates">
        {{ clock.dates }}&nbsp;&nbsp;星期{{ clock.week }}
      </p>
    </template>
    <!-- 番茄钟 -->
    <template v-else>
      <div class="tomato-progress">
        <svg width="250px" height="250px" class="svg">
          <circle
            r="100"
            cy="125"
            cx="125"
            stroke-width="16"
            stroke="#ff9a86"
            stroke-linejoin="round"
            stroke-linecap="round"
            fill="none"
          />
          <circle
            class="progress"
            r="100"
            cy="125"
            cx="125"
            stroke-width="16"
            stroke="#ffffff"
            stroke-linejoin="round"
            stroke-linecap="round"
            fill="none"
            stroke-dashoffset="0"
          />
        </svg>
        <h3 :class="['tomato-time', { shadow: tomato.start }]">
          <span class="hour">{{ tomato.minute }}</span>
          <em>:</em>
          <span class="minute">{{ tomato.second }}</span>
        </h3>
      </div>
      <button
        type="button"
        :class="['tomato-start', { shadow: tomato.start }]"
        @click="operate('tomato-button')"
      >
        {{ !tomato.start ? "开始" : "放弃" }}专注
      </button>
    </template>
  </div>
</template>

<script>
import "../assets/fonts/iconfont.css";
import { ref, reactive, onBeforeMount, nextTick } from "vue";
export default {
  name: "clock",
  setup(props, context) {
    const skin = ref(1); //皮肤，1-暗黑，2-浅色
    const mode = ref(1); //模式，1-时钟，2-番茄钟
    const weekMap = {
      0: "日",
      1: "一",
      2: "二",
      3: "三",
      4: "四",
      5: "五",
      6: "六",
    }; //星期map
    let timer = null; //计时器
    const clock = reactive({
      breath: false, //是否开启冒号呼吸动画
      hour: "00",
      minute: "00",
      dates: "2020/01/01",
      week: "一",
    });
    const tomato = reactive({
      start: false, //是否启动
      duration: 25, //时长，默认25分钟
      remain: "", //剩余时间
      minute: "00", //分钟
      second: "00", //秒
      path: null, //进度路径
      lineLength: null, //周长
      init() {
        this.path = document.querySelector(".progress");
        this.lineLength = 2 * Math.PI * 100; //计算周长
        this.path.setAttribute(
          "stroke-dasharray",
          `${this.lineLength} ${this.lineLength}`
        ); //绘制描边
        //初始化时间
        this.remain = this.duration * 60;
        const m = Math.floor(tomato.remain / 60);
        const s = tomato.remain % 60;
        this.minute = fixZero(m);
        this.second = fixZero(s);
        // 插入提示音
        if (!document.querySelector("#tomato-sound")) {
          const audio = document.createElement("audio");
          audio.src = "./ding.wav";
          audio.id = "tomato-sound";
          audio.preload = true;
          document.body.appendChild(audio);
        }
      },
      play() {
        document.querySelector("#tomato-sound").play();
      },
      reset() {
        this.start = false;
        clearTimeout(timer);
        this.path.setAttribute("stroke-dashoffset", "0");
        this.remain = this.duration * 60;
        this.minute = fixZero(this.duration);
        this.second = "00";
      },
    });

    // 操作，[type]skin-切换皮肤,tomato-番茄钟,tomato-button-番茄钟按钮
    const operate = (type) => {
      if (type === "tomato") {
        clearTimeout(timer);
        mode.value = mode.value === 1 ? 2 : 1;
        nextTick(() => {
          if (mode.value === 1) {
            tomato.reset();
            document.querySelector("#tomato-sound").remove();
            counter();
          } else {
            tomato.init();
          }
        });
      } else if (type === "tomato-button") {
        if (!tomato.start) {
          tomato.start = true;
          counter();
        } else {
          tomato.reset();
        }
      } else if (type === "skin") {
        skin.value = skin.value === 1 ? 2 : 1;
        localStorage.setItem("clock-skin", skin.value);
      }
    };

    // 补零
    function fixZero(data) {
      return data < 10 ? `0${data}` : data;
    }

    // 计时器
    const counter = () => {
      if (mode.value === 1) {
        const now = new Date();
        clock.hour = fixZero(now.getHours());
        clock.minute = fixZero(now.getMinutes());
        clock.dates = `${now.getFullYear()}/${fixZero(
          now.getMonth() + 1
        )}/${fixZero(now.getDate())}`;
        clock.week = weekMap[now.getDay()];
        timer = setTimeout(counter, 1000);
      } else {
        const m = Math.floor(tomato.remain / 60);
        const s = tomato.remain % 60;
        tomato.minute = fixZero(m);
        tomato.second = fixZero(s);
        // 计算时间进度并和进度条绑定
        const percent = tomato.remain / (tomato.duration * 60);
        tomato.path.setAttribute(
          "stroke-dashoffset",
          tomato.lineLength * (1 - percent)
        );
        tomato.remain--;
        if (tomato.remain >= 0) {
          timer = setTimeout(counter, 1000);
        } else {
          tomato.play();
          tomato.reset();
        }
      }
    };

    // 切换全屏
    const screenUtils = {
      enterFullscreen(element) {
        element = element || document.documentElement;
        if (element.requestFullscreen) {
          element.requestFullscreen();
        } else if (element.mozRequestFullScreen) {
          element.mozRequestFullScreen();
        } else if (element.webkitRequestFullscreen) {
          element.webkitRequestFullscreen();
        } else if (element.msRequestFullscreen) {
          element.msRequestFullscreen();
        }
      },
      exitFullscreen() {
        if (document.exitFullscreen) {
          document.exitFullscreen();
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen();
        } else if (document.webkitExitFullscreen) {
          document.webkitExitFullscreen();
        } else if (document.msCancelFullScreen) {
          document.msCancelFullScreen();
        }
      },
    };
    const toogleFullscreen = (element) => {
      const isFull = !!(
        document.fullscreenElement ||
        document.webkitFullscreenElement ||
        document.mozFullScreenElement ||
        document.msFullscreenElement
      );
      screenUtils[`${!isFull ? "enter" : "exit"}Fullscreen`]();
    };

    onBeforeMount(() => {
      // 读取本地保存的模式
      const localskin = localStorage.getItem("clock-skin");
      if (localskin) {
        skin.value = Number(localskin);
      }
      // 获取当前时间
      counter();
    });

    return {
      skin,
      mode,
      clock,
      tomato,
      counter,
      operate,
    };
  },
};
</script>

<style lang="less" scoped>
@black: #141414;
@light: #fff;
@tomato: #ff684b;
.clock-app {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
  -webkit-user-select: none;
  background: @black;
  transition: background-color 1s ease;
  &.light {
    background: @light;
    .clock-hours {
      color: #333;
      em {
        color: #333;
      }
    }
    .clock-dates {
      color: #666;
    }
    .ico {
      color: #666;
      &:hover {
        color: #686868;
      }
    }
  }
  &.tomato {
    background: @tomato;
    .ico {
      color: #eee !important;
      &:hover {
        color: #fff !important;
      }
    }
  }
  .tools-bar {
    position: absolute;
    top: 1rem;
    right: 1rem;
    z-index: 4;
    .ico {
      cursor: pointer;
      margin-left: 10px;
      font-size: 16px;
      color: #a3a3a3;
      &:hover {
        color: #d2d2d2;
      }
    }
  }
  // @keyframes breath {
  //   from {
  //     opacity: 1;
  //   }
  //   to {
  //     opacity: 0;
  //   }
  // }
  .clock-hours {
    position: relative;
    margin-bottom: 20px;
    text-align: center;
    font-family: system-ui;
    line-height: 1;
    word-spacing: -24px;
    font-size: 28vw;
    color: #fff;
    transition: color font-size 1s ease;
    span,
    em {
      display: inline-block;
      font-style: normal;
      vertical-align: middle;
      transition: color font-size 1s ease;
    }
    em {
      position: relative;
      top: -3vw;
      // text-shadow: 5px 5px 5px #fc2f2f;
    }
    // .hour {
    //   text-shadow: 5px 5px 5px #3cd9a0;
    // }
    // .minute {
    //   text-shadow: 5px 5px 5px #fc2f2f;
    // }
    // .breath {
    //   animation: breath 1s ease infinite;
    //   transition: color font-size 1s ease;
    // }
  }
  .clock-dates {
    font-family: sans-serif;
    font-size: 6vw;
    color: #dedede;
    transition: color font-size 1s ease;
  }

  .tomato-progress {
    position: relative;
    margin-bottom: 20px;
    svg {
      transition: stroke-dashoffset 0.6s ease 0s, stroke 0.6s ease 0s;
      transform: rotate(-90deg);
      transform-origin: 50% 50%;
    }
  }
  .tomato-time {
    position: absolute;
    z-index: 2;
    width: 100%;
    top: 50%;
    transform: translate3d(0, -50%, 0);
    font-size: 50px;
    color: #fff;
    span,
    em {
      display: inline-block;
      font-style: normal;
      vertical-align: middle;
    }
    em {
      position: relative;
      top: -5px;
    }
    &.shadow {
      span,
      em {
        text-shadow: 5px 5px 5px #f33d3d;
      }
    }
  }
  .tomato-start {
    display: inline-block;
    width: 200px;
    padding: 10px 0;
    font-size: 16px;
    color: #fff;
    outline: none;
    background: none;
    border: 2px solid #fff;
    border-radius: 50px;
    &.shadow {
      color: #d63e21;
      border: 2px solid #e25034;
    }
  }
}
@media screen and (max-width: 1024px) {
  .clock-app {
    &.light {
      .ico {
        color: #666;
        &:hover {
          color: #666;
        }
      }
    }
    &.tomato {
      .ico {
        color: #eee;
        &:hover {
          color: #eee;
        }
      }
    }
    .tools-bar {
      .ico {
        color: #a3a3a3;
        &:hover {
          color: #a3a3a3;
        }
      }
    }
  }
}
</style>