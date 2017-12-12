# 欢迎入坑 Vue

Hi~ o(_￣ ▽ ￣_)ブ

## 入坑导言

说实话 Vue 的概念挺多的，而且[官方文档](https://cn.vuejs.org/v2/guide/installation.html)解释的肯定比我详细，比我理解的深... ⁄(⁄ ⁄•⁄ω⁄•⁄ ⁄)⁄

想要这次的分享让大家收获比较多的东西，我打算从两个示例入手。

第一个，目的是入门，展示下如何编写一个 Vue 组件；

第二个，稍微增加下条件，看看使用 Vue 的思维，如何构建一个项目。

下面是这两个示例想要表达的 Vue 核心概念，我们先不用纠结这些概念，看完两个例子，我们再回头来看。

## 一个.vue 文件组件

### template

1. 模板语法

2. 指令

   v-bind、v-on、v-if、v-show

### script

1. 何为 MVVM ？

1. [生命周期](https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)

   (1) created

   (2) mounted

   (3) updated

   (4) destroy

1. [数据跟踪（响应式原理）](https://cn.vuejs.org/v2/guide/reactivity.html)

   (1) data

   (2) computed

   (3) watch

1. 单向数据流

   (1) props

   (2) $emit

## 例子 1：倒计时

1. 描述：

   现有一个比赛（嗯... 不知道啥比赛），比赛时间为 60s，比赛时间减为 0 时，比赛结束。

2. 代码：

   (1) template

   ```html
    <template>
    <div id="counting-down">
    <!-- {{ x }} “Mustache”语法 (双大括号) -->
    <!-- Mustache 标签将会被替代为对应数据对象上 remainTime 属性的值。无论何时，绑定的数据对象上 remainTime 属性发生了改变，插值处的内容都会更新。 -->
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
        //生命周期，装载。当template最外层，装载到dom上时触发，不能保证子组件都装载。我们这个例子没有子组件，就可以把这个生命钩子当做，dom已经准备好了。
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

   ![vue响应式原理](/static/data.png)

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

   Vue 的单项数据流规定

        当父组件的属性变化时，将传导给子组件，但是反过来不会。这是为了防止子组件无意间修改了父组件的状态，来避免应用的数据流变得难以理解

   由于裁判需要控制时间的能力，且裁判和倒计时显然没有什么父子联系。这时比赛剩余时间放在倒计时组件中控制的话，我们就没有什么好办法能让裁判很容易的控制它。所以 remainTime 需要上提到比赛这个父组件中，这时父组件变可通过 v-on 赋予裁判调整比赛剩余时间的权力。

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
               //当调整时间的button被点击是，触发比赛组件定义在当前组件上的assignTime事件，注入time参数，触发回调
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

## More

1. [状态管理](https://cn.vuejs.org/v2/guide/state-management.html)

   Vuex

   由于多个状态分散的跨越在许多组件和交互间各个角落，大型应用复杂度也经常逐渐增长。为了解决这个问题，Vue 提供 vuex

   ![state](/static/state.png)

2. 配置开发工具

   以 [vscode](https://code.visualstudio.com/) 为例

   推荐安装插件 Vetur

   格式化工具 Prettier

   语法检查 eslint

3. [webpack](https://webpack.js.org/)

   前端打包工具。 weex 中 JS Bundle 的形成就靠它。

   将通过自定义配置文件，将你写的源码编译组合成你想要的样子。

   在当前项目目录运行下面的命令行，webpack 根据配置文件  将源码打包为可以上线的最终版本(解析.vue 文件，抽离成 js css 文件；压缩混淆 js 代码；ES6 代码降级成 ES5 等等)。

   ```bash
    npm run build
   ```
