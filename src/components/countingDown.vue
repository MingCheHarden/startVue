<template>
  <div class="counting-down">
    <span>比赛总时间: {{totalTime}}</span>
    <span>剩余时间: {{remainTime}}</span>
  </div>
</template>

<script>
export default {
    name: "countingDown",
    data() {
        return {};
    },
    props: ["totalTime", "remainTime", "matchState"],
    computed: {},
    watch: {},
    mounted() {},
    methods: {}
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
