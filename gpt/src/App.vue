<template>
  <div class="chat-container">
    <div class="messages" v-for="message in messages" :key="message.id">
      <div class="message" :class="{ 'user': message.sender === 'user', 'chatbot': message.sender === 'chatbot' }">
        {{ message.sender  }} : {{ message.text }}
      </div>
    </div>
    <div class="input-container">
      <input type="text" v-model="prompt" @keyup.enter="getResponse" placeholder="Enter your message here"
        class="input-area">
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      prompt: "",
      messages: [],
      presets: {
        chat: {
        }
      }
    }
  },
  methods: {
    async getResponse() {
      this.messages.push({ id: Date.now(), sender: "user", text: this.prompt }); 
      const apiKey = "YOUR_API_KEY";
      const response = await fetch('https://api.openai.com/v1/completions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${apiKey}`
        },
        body: JSON.stringify({
          "model": "text-davinci-003",
          "prompt": this.prompt,
          "temperature": 0.5,
          "max_tokens": 60,
          "top_p": 0.3,
          "frequency_penalty": 0.5,
          "presence_penalty": 0
        })
      });
      this.prompt = "";
      const jsonResponse = await response.json();
      this.messages.push({ id: Date.now(), sender: "chatbot", text: jsonResponse.choices[0].text });
      console.log(this.messages)
    }
  }
}
</script>

