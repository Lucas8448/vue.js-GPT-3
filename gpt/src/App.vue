<template>
  <div class="chat-container">
    <div class="input-container">
      <input type="text" v-model="prompt" @keyup.enter="getResponse" placeholder="Enter your message here">
      <button @click="getResponse">Send</button>
    </div>
    <div class="messages" v-for="message in messages" :key="message.id">
      <div class="message" :class="{ 'user': message.sender === 'user', 'chatbot': message.sender === 'chatbot' }">
        {{ message.sender  }} : {{ message.text }}
      </div>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      prompt: "",
      messages: []
    }
  },
  methods: {
    async getResponse() {
      const maxTokens = 200;
      const apiKey = "YOUR_API_KEY";
      const response = await fetch('https://api.openai.com/v1/engines/davinci/completions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${apiKey}`
        },
        body: JSON.stringify({ prompt: this.prompt, max_tokens: maxTokens })
      });
      const jsonResponse = await response.json();
      this.messages.unshift({ id: Date.now(), sender: "user", text: this.prompt });
      this.messages.unshift({ id: Date.now(), sender: "chatbot", text: jsonResponse.choices[0].text });
      this.prompt = "";
    }
  }
}
</script>