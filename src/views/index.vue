<template>
  <div class="wrapper" v-loading.fullscreen.lock="loading">
    <section class="content">
      <div class="result-display">
        <div v-if="result">你输入的内容是：{{ result }}</div>
      </div>
    </section>
    <section class="input">
      <el-input v-model="msg" type="textarea" :rows="5"></el-input>
      <el-button type="primary" @click="sendMsg">发送</el-button>
    </section>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const loading = ref(false)
const msg = ref('')
const result = ref('')

function sendMsg() {
  console.log(msg.value)
  result.value = msg.value
  msg.value = ''
}
</script>

<style scoped>
.wrapper {
  height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  /* 重置父级的限制 */
  position: fixed;
  top: 0;
  left: 0;
}
.content {
  background-color: green;
  color: white;
  flex: 1;
  overflow-y: auto; /* 如果内容太多可以滚动 */
}

.result-display {
  white-space: pre-wrap; /* 保留换行和空格 */
  word-wrap: break-word; /* 长单词自动换行 */
}


.input{
  position: relative;
  background-color: #f5f5f5;
  padding: 20px;
}

.el-button {
  position: absolute;
  bottom: 25px;
  right: 25px;
}
</style>
