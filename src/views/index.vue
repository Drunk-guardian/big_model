<template>
  <div class="wrapper" v-loading.fullscreen.lock="loading">
    <section class="content">
      <div class="result-display">
        <pre class="msg-box">{{ userInput }}</pre>
        <pre class = "think">{{ think }}</pre>
        <pre>{{ result }}</pre>
      </div>
    </section>
    <section class="input">
      <el-input 
        v-model="msg" 
        type="textarea" 
        :rows="5"
        @keydown="handleKeyDown"
        placeholder="(Enter发送，Ctrl+Enter换行)"
      ></el-input>
      <el-button type="primary" @click="sendMsg">发送</el-button>
    </section>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const loading = ref(false)
const think = ref('')
const msg = ref('')
const result = ref('')
const userInput = ref('') // 用于显示用户输入内容的变量

// 处理键盘按键事件
function handleKeyDown(event) {
  
  // 检查是否按下了 Enter 键
  if (event.key === 'Enter') {
    // 如果同时按下了 Ctrl 键，主动插入换行符
    if (event.ctrlKey) {
      event.preventDefault() // 阻止默认行为
      
      // 获取当前光标位置
      const textarea = event.target
      const start = textarea.selectionStart
      const end = textarea.selectionEnd
      
      // 在光标位置插入换行符
      const newValue = msg.value.substring(0, start) + '\n' + msg.value.substring(end)
      msg.value = newValue
      
      // 将光标移动到换行符后面
      setTimeout(() => {
        textarea.selectionStart = textarea.selectionEnd = start + 1
        textarea.focus()
      }, 0)
    } else {
      // 只按 Enter 键，阻止默认换行行为并发送消息
      event.preventDefault()
      sendMsg()
    }
  }
}

function sendMsg() {
  // 保存用户输入内容用于显示
  userInput.value = msg.value

  let data ={
    "model":"Qwen/QwQ-32B",
    "messages":[
      {
        "role":"user",
        "content":msg.value
      }
    ],
    "stream":true
  }
  loading.value = true
  
  axios.request({
    url: 'https://api.siliconflow.cn/v1/chat/completions',
    method: 'post',
    headers: { 
      'Content-Type': 'application/json', 
      'Accept': 'application/json', 
      'Authorization': 'Bearer sk-dhyofqmlqevepadtfbjjmtvelluvgoqixawhgqcyhmiysdtl'
    },
    data: JSON.stringify(data)
  }).then(response => {
    console.dir(response)
    // 将结果清空
    result.value = ''
    msg.value = ''
    // 流式输出返回的是一个长字符串，我们先按照两个回车将字符串截断为数组
    const array = response.data.split('\n\n')
    // 循环数组
    for (let i in array) {
      // 如果是空白行，不做任何处理
      if (array[i] === '') {
        continue
      }
      // 将每行的字符串 去掉data: (前6个字符)
      const str = array[i].substring(6)
      // 如果截断后 字符是 [DONE] 表示 已经说完了
      if (str === '[DONE]') {
        break
      }
      // 将字符串转换成JSON
      const json = JSON.parse(str)
      setTimeout(() => {
        if (json.choices[0].delta.reasoning_content) {
          think.value += json.choices[0].delta.reasoning_content
        }
        if (json.choices[0].delta.content) {
          result.value += json.choices[0].delta.content
        }
      }, 50 * i)
    }
  }).catch(error => {
    console.dir(error)
  }).finally(() => {
    loading.value = false
  })
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

.msg-box {
  width: fit-content; /* 宽度适应内容 */
  background-color: rgb(13, 13, 240);
  color: white;
  margin-bottom: 20px;
  border-radius: 5px;
  float: right; /* 右浮动实现右对齐 */
  max-width: 50%; /* 限制最大宽度 */
}

.content {
  background-color: #f5f5f5;
  color: black;
  flex: 1;
  overflow: auto; /* 如果内容太多可以滚动 */
  padding: 20px; /* 添加内边距 */
}

.result-display {
  max-width: 100%; /* 限制最大宽度 */
  overflow-wrap: break-word; /* 强制长单词换行 */
  word-break: break-word; /* 在任意字符间换行 */
  white-space: pre-wrap; /* 保留换行和空格，但允许换行 */
  hyphens: auto; /* 自动添加连字符 */
}

.result-display pre {
  padding: 0;
  font-family: inherit; /* 使用系统字体而不是等宽字体 */
  white-space: pre-wrap; /* 保留格式但允许换行 */
  word-wrap: break-word; /* 长单词强制换行 */
  overflow-wrap: break-word; /* 现代浏览器的强制换行 */
  max-width: 50%; /* 回答框占容器宽度的一半 */
  margin-left: 50px;
  box-sizing: border-box; /* 包含padding和border在宽度内 */
}

.think {
  opacity: 0.7;
  font-style: italic;
  color: #666;
  margin-bottom: 10px; /* 添加底部间距 */
  padding: 20%; /* 添加内边距 */
  background-color: #d8d5d5; /* 添加背景色区分 */
  border-radius: 5px; /* 圆角 */
}

.input {
  position: relative;
  background-color: #f5f5f5;
  padding: 20px;
  border-top: 1px solid #d42d2d; /* 添加分割线 */
}

.el-button {
  position: absolute;
  bottom: 25px;
  right: 25px;
}

/* 确保输入框也不会出现横向滚动 */
.el-input {
  width: calc(100% - 100px); /* 为按钮留出空间 */
}


.think, .result-display pre:not(.msg-box) {
  clear: both; /* 清除浮动 */
  margin-top: 10px; /* 增加顶部间距 */
}

</style>

