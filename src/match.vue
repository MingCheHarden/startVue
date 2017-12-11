<template>
  <div id="match">
    <countingDown 
      :totalTime=totalTime
      :remainTime=remainTime
      :matchState="state" />
    <judge
      v-on:assignTime="assignTime"
      v-on:matchControl="matchControl" />
  </div>
</template>

<script>
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
            this.state = "stopped";
        },
        startCounting() {
            this.timer = setInterval(() => {
                this.remainTime--;
            }, 1000);
            this.state = "running";
        }
    },
    watch: {
        judgeOrder(val) {
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
        countingDown,
        judge
    }
};
</script>

<style>
#app {
    font-family: "Avenir", Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
</style>
