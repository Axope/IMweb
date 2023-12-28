<template>
  <div>
    <h1>Home Page</h1>
    <div>
      <label for="to">To:</label>
      <input type="text" id="to" v-model="to" required>
    </div>
    <div class="type-section">
      <label>Type:</label>
      <div>
        <label :for="userType">
          <input type="radio" v-model="selectedOption" :value="userType" :id="userType">
          To User
        </label>
        <label :for="groupType">
          <input type="radio" v-model="selectedOption" :value="groupType" :id="groupType">
          To Group
        </label>
      </div>
    </div>
    <div>
      <label for="content">Content:</label>
      <textarea id="content" v-model="content" required></textarea>
    </div>
    <button @click="sendMessage">Send Message</button>
    <div>
      <h2>Received Message:</h2>
      <p>{{ receivedMessage }}</p>
    </div>
  </div>
</template>

<script>
import { encodeMessage, decodeMessage } from '@/protocol/message.js';

export default {
  data() {
    return {
      selectedOption: 'ToUser',
      to: '',
      content: '',
      receivedMessage: '',
      userType: 'ToUser', // 用户选项的值
      groupType: 'ToGroup', // 用户组选项的值
    };
  },
  mounted() {
    this.connectWebSocket();
  },
  methods: {
    connectWebSocket() {
      const uuid = localStorage.getItem('uuid');
      if (!uuid) {
        this.$router.push('/login');
        return;
      }

      // 建立WebSocket连接
      this.$data.socket = new WebSocket(`ws://localhost:9876/socket?uuid=${uuid}`);
      this.$data.socket.onopen = () => {
        console.log('WebSocket connected');
      };
      this.$data.socket.onerror = (error) => {
        console.error('WebSocket error:', error);
      };
      this.$data.socket.onmessage = (event) => {
        // 处理接收到的消息
        let reader = new FileReader();
        reader.readAsArrayBuffer(event.data);
        reader.onload = (
          (event) => {
            let messagePB = decodeMessage(new Uint8Array(event.target.result))
            this.receivedMessage = JSON.stringify(messagePB, null, 2);
          }
        )
      };
    },
    sendMessage() {
      const from = localStorage.getItem('uuid');
      if (!from) {
        this.$router.push('/login');
        return;
      }

      // 构建消息对象
      const message = {
        from,
        to: this.to,
        type: this.selectedOption,
        content: this.content,
      };

      console.log(message)
      // 序列化消息对象
      const encodedMessage = encodeMessage(message);

      // 发送消息到后端
      this.$data.socket.send(encodedMessage);

      // // 清空输入框
      // this.to = '';
      // this.type = 0;
      // this.content = '';
    },
  },
};
</script>

<style>
.type-section {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.type-section label {
  margin-right: 10px;
}
</style>