<template>
  <div class="judge">
    <span>勒布朗·三井寿</span>
    <img src="../assets/judge.jpg" >
    <button class="startBtn" @click="controlBtnClick('start')">开始</button>
    <button class="stopBtn" @click="controlBtnClick('stop')">暂停</button>
    <input v-model="message" placeholder="edit me">
    <button class="assignTimeBtn" @click="assignTime(message)">调整时间为{{ message }}</button>
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
.judge {
    position: relative;
    height: 100px;
    margin-top: 50px;
    background: #ccc;
}
button {
    background: #ccc;
}
img {
    width: 100px;
    height: 100px;
    border-radius: 100%;
    position: absolute;
    top: 0;
    left: 0;
}
span {
    position: absolute;
    bottom: 0;
    z-index: 5;
    color: #fff;
}
.startBtn {
    position: absolute;
    top: 10px;
    left: 150px;
}
.stopBtn {
    top: 40px;
    left: 150px;
    position: absolute;
}
input {
    top: 72px;
    left: 150px;
    position: absolute;
}
.assignTimeBtn {
    top: 72px;
    left: 290px;
    position: absolute;
}
</style>

