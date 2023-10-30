<script setup>
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";

const socket = io('http://localhost:3001');

const messages = ref([]);
const messageText = ref('');
const joined = ref(false);
const name = ref('');
const typingDisplay = ref('');

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) =>{
    messages.value = response;
  });

  socket.on('message', (message) => {
    messages.value.push(message);
  })

  socket.on('typing', ({ name, isTyping }) => {
    if(isTyping){
      typingDisplay.value += `${name} is typing... `;
    } else {
      typingDisplay.value = '';
    }
  })
   
  
})

const join = ()  => {
  socket.emit('join', { name: name.value }, () => {
    joined.value = true;
  })
}

const sendMessage = () => {
  socket.emit('createMessage', { text: messageText.value }, () => {
    messageText.value = '';
  })
}

let timeout;
const emitTyping = () => {
  socket.emit('typing', { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit('typing', {isTyping: false});
  }, 2000);

}

</script>

<template>
  <div class="chat">
    <div v-if="!joined" class="join-form">
      <form @submit.prevent="join">
        <label for="name">What's your name?</label>
        <input v-model="name" id="name" />
        <button type="submit">Join</button>
      </form>
    </div>

    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages" :key="message.id" class="message">
          <span class="message-sender">[{{ message.name }}]:</span>
          <span class="message-text">{{ message.text }}</span>
        </div>
      </div>

      <div class="typing-indicator" v-if="typingDisplay">{{ typingDisplay }}</div>
      
      <div class="message-input">
         <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="emitTyping" />
          <button type="submit">Send</button>
         </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .chat {
    font-family: Arial, sans-serif;
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    text-align: center; /* Center the content horizontally */
  }

  .join-form {
    text-align: center;
  }

  label {
    font-weight: bold;
  }

  input {
    width: 100%;
    padding: 8px;
    margin: 8px 0;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  button {
    background-color: #4CAF50;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  button:hover {
    background-color: #45a049;
  }

  .chat-container {
    border: 1px solid #ccc;
    padding: 10px;
    border-radius: 5px;
  }

  .messages-container {
    margin-top: 10px;
  }

  .message {
    margin: 5px 0;
  }

  .message-sender {
    font-weight: bold;
  }

  .typing-indicator {
    font-style: italic;
    color: #aaa;
  }
</style>
