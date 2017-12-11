# 欢迎入坑 Vue

Hi~ o(_￣ ▽ ￣_)ブ

## 入坑导言

    说实话Vue的概念挺多的，而且官方文档解释的肯定比我详细，比我理解的深... ⁄(⁄ ⁄•⁄ω⁄•⁄ ⁄)⁄
    想要这次的分享让大家收获比较多的东西，我打算从两个示例入手。第一个，目的是入门，展示下如何编写一个Vue组件；第二个，稍微增加下条件，看看使用Vue的思维，如何构建一个项目。

    下面是这两个示例，想要表达的核心概念，先顺一眼，带着疑问。

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

## 例子 1：倒计时

1. 描述：

   现有一个比赛（嗯... 不知道啥比赛），比赛时间为 60s，比赛时间减为 0 时，比赛结束。

2. 代码：

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

3. 知识点

   （1） 响应式系统

   data 中的 remainTime，在 Vue 实例化的时候会被监听 setter 和 getter。也就是说，在 template 中使用了 remainTime，这时 getter 操作会将 remainTime 记录为 template 的依赖；而在脚本改变 remainTime 值时（setInterval），setter 操作发起通知，vue 接收到通知，便根据 getter 收集到的依赖树，通知相应的依赖更新，再触发 template 的重新渲染。

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
