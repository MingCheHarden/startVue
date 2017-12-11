<template>
  <div id="counting-down">
      <h1>比赛剩余时间:{{remainTime}}</h1>
  </div>
</template>

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
