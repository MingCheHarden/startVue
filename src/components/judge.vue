<template>
  <div class="judge">
    <span>姓名：勒布朗·三井寿</span>
    <button @click="controlBtnClick('start')">开始</button>
    <button @click="controlBtnClick('stop')">暂停</button>
    <input v-model="message" placeholder="edit me">
    <button @click="assignTime(message)">调整时间为： {{ message }}</button>
  </div>
</template>

<script>
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
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
