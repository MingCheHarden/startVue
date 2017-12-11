# Vue 分享

## 前端基础

1. HTML

   (1) 语法不区分大小写

   (2)

2. CSS

   (1) flex

   (2) 盒模型

3. JavaScript

## 一个.vue 文件组件

### template

1. 模板语法

2. 条件渲染与列表渲染

3. 指令

   v-bind、v-on

### script

1. 何为 MVVM ？

1. 生命周期

1. 数据跟踪（响应式原理）

   (1) data

   (2) computed

   (3) watch

1. 单向数据流，状态管理

   (1) props

   (2) $emit

   (3) Vuex

### style

## 例子 1：倒计时

1. 描述：

   现有一个比赛（嗯... 不知道啥比赛），比赛时间为 60s，比赛时间减为 0 时，比赛结束。

2. 页面： ./countingDown.html

3. 代码：

   (1) template

   ```html
    <template>
    <div id="counting-down">
        <h1>比赛剩余时间:{{remainTime}}</h1>
    </div>
    </template>
   ```

   (2) script

   ```js
   <script>
   export default {
    data() {
        return {
            remainTime: 60
        };
    },
    methods: {},
    mounted() {
        setInterval(() => {
            this.remainTime > 0 ? this.remainTime-- : 0;
        }, 1000);
    },
    components: {}
   };
   </script>
   ```

4. 知识点

   （1） 响应式系统

          data中的remainTime，在vue实例化的时候会被监听setter和getter。
          也就是说，在template中使用了remainTime，这时getter操作会将remainTime记录为template的依赖；而在脚本改变remainTime值时（setInterval），setter操作发起通知，vue接收到通知，便根据getter收集到的依赖树，通知相应的依赖更新，再触发template的重新渲染。

## 例子 2

1. 描述：

   例子 1 主要是为了给大家介绍，Vue 最核心的响应式机制，就是 Model->View，数据层怎么和视图层建立的联系。

   接下来我们添加一个新人物进来：裁判。看看这个时候我们面临什么问题。

   先定义一下裁判的功能：

   （1）可以开始、暂停比赛。

   （2）当比赛进程错误时，需要将比赛拨回到正确的道路上，比如回表，重新赋值倒计时时间。

2. 分析一波

   (1) 构建组件，确认关系。

       父: 比赛
       子: 倒计时+裁判

   (2) 建立 Model，并且定义数据的流动。

   比赛状态、倒计时时间

   Vue 的单项数据流规定

        当父组件的属性变化时，将传导给子组件，但是反过来不会。这是为了防止子组件无意间修改了父组件的状态，来避免应用的数据流变得难以理解

   由于裁判需要控制时间的能力，且裁判和倒计时显然没有什么父子联系。 这时比赛剩余时间放在倒计时组件中控制的话，我们就没有什么好办法能让裁判很容易的控制它。所以 remainTime 需要上提到比赛这个父组件中，这时父组件变可通过 v-on 赋予裁判调整比赛剩余时间的权力。

   ![数据流动](/static/model-flow.jpeg)

3. 代码

   Match.vue

   (1) template

   ```html
   <template>
   <div id="match">
       <!-- 子组件。标签名一般都使用驼峰法 -->
       <countingDown
       v-bind:totalTime=totalTime
       v-bind:remainTime=remainTime
       v-bind:matchState="state" />
       <!--  totalTime,remainTime,matchState会通过props传递给countingDown组件，props的值和data一样，响应式变化  -->
       <judge
       v-on:assignTime="assignTime"
       v-on:matchControl="matchControl" />
       <!-- assignTime，matchControl为父组件给judge组件的自定义事件，可以把它们理解成父组件赋予子组件一些特权，子组件便能做一些不在它们管辖范围外的事。但这一切的变化，都是在父组件内做。 -->
   </div>
   </template>
   ```

   (2) script

   ```js
   //引入子组件，驼峰法，要在Vue实例中使用还缺少一步注册
   import countingDown from "./components/countingDown";
   import judge from "./components/judge";

   export default {
     name: "match",
     data() {
       return {
         state: "stopped", // running | stopped | runningOut
         judgeOrder: "stop", //start | stop
         totalTime: 60,
         remainTime: 60,
         timer: null
       };
     },
     methods: {
       assignTime(time) {
         this.remainTime = time;
       },
       matchControl(state) {
         this.judgeOrder = state;
       },
       stopCounting() {
         clearInterval(this.timer);
       },
       startCounting() {
         this.timer = setInterval(() => {
           this.remainTime--;
         }, 1000);
       }
     },
     watch: {
       judgeOrder(val) {
         // val为judgeOrder目前的值
         // 监听judgeOrder的变化，并且可以在变化时做一些事情...
         if (val === "start" && this.state === "stopped") {
           this.startCounting();
         }
         if (val === "stop" && this.state === "running") {
           this.stopCounting();
         }
       },
       remainTime(val) {
         if (val === 0) {
           this.stopCounting();
         }
       }
     },
     components: {
       // 在该组件中，注册子组件
       countingDown,
       judge
     }
   };
   ```

   judge.vue

   (1) template

   ```html
   <div class="judge">
       <span>姓名：勒布朗·三井寿</span>
       <!-- @click是在这个元素上注册点击事件 -->
       <button @click="controlBtnClick('start')">开始</button>
       <button @click="controlBtnClick('stop')">暂停</button>
       <input v-model="message" placeholder="edit me">
       <button @click="assignTime(message)">调整时间为： {{ message }}</button>
   </div>
   ```

   (2) script

   ```js
   export default {
     name: "judge",
     data() {
       return {
         message: ""
       };
     },
     methods: {
       assignTime(time) {
         this.$emit("assignTime", time);
       },
       controlBtnClick(state) {
         this.$emit("matchControl", state);
       }
     }
   };
   ```

   countingDown.vue

   (1) template

   ```html
   <div class="counting-down">
    <span>比赛总时间: {{totalTime}}</span>
    <span>剩余时间: {{remainTime}}</span>
   </div>
   ```

   (2) script

   ```js
   export default {
     name: "countingDown",
     data() {
       return {};
     },
     props: ["totalTime", "remainTime", "matchState"] //从父组件传过来的值
   };
   ```
