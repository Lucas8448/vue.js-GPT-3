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
    <select class="chatbot-dropdown" v-model="selectedPreset">
      <option v-for="preset in presets" :value="preset">{{ preset.name }}</option>
    </select>
  </div>
</template>


<script>
export default {
  data() {
    return {
      prompt: "",
      messages: [],
      selectedPreset: null,
      presets: [
        {
          name: "Chat",
          url: 'https://api.openai.com/v1/completions',
          body: JSON.stringify({
            "model": "text-davinci-003",
            "prompt": this.prompt,
            "temperature": 0.9,
            "max_tokens": 150,
            "top_p": 1,
            "frequency_penalty": 0,
            "presence_penalty": 0.6,
            "stop": [" Human:", " AI:"]
          })
        },
        {
          name: "Directions",
          url: 'https://api.openai.com/v1/completions',
          body: JSON.stringify({
            "model": "text-davinci-003",
            "prompt": this.prompt,
            "temperature": 0.3,
            "max_tokens": 64,
            "top_p": 1.0,
            "frequency_penalty": 0.0,
            "presence_penalty": 0.0
          })
        },
        {
          name: "Outline Essay",
          url: 'https://api.openai.com/v1/completions',
          body: JSON.stringify({
            "model": "text-davinci-003",
            "prompt": this.prompt,
            "temperature": 0.3,
            "max_tokens": 150,
            "top_p": 1,
            "frequency_penalty": 0,
            "presence_penalty": 0
          })
        },
        {
          name: "Explain Code",
          url: 'https://api.openai.com/v1/completions',
          body: {
            "model": "code-davinci-002",
            "prompt": this.prompt,
            "temperature": 0,
            "max_tokens": 64,
            "top_p": 1.0,
            "frequency_penalty": 0.0,
            "presence_penalty": 0.0,
            "stop": ["\"\"\""]
          }
        },
        {
          name: "Sarcasm",
          url: 'https://api.openai.com/v1/completions',
          body: JSON.stringify({
            "model": "text-davinci-003",
            "prompt": this.prompt,
            "temperature": 0.5,
            "max_tokens": 60,
            "top_p": 0.3,
            "frequency_penalty": 0.5,
            "presence_penalty": 0.0
          })
        }
      ],
    }
  },
  mounted() {
    this.selectedPreset = this.presets[0]
  },
  methods: {
    async getResponse() {
      this.messages.push({ id: Date.now(), sender: "user", text: this.prompt }); 
      const apiKey = "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";
      const response = await fetch(this.selectedPreset.url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${apiKey}`
        },
        body: this.selectedPreset.body
      });
      console.log(this.selectedPreset)
      this.prompt = "";
      const jsonResponse = await response.json();
      this.messages.push({ id: Date.now(), sender: "chatbot", text: jsonResponse.choices[0].text });
      console.log(this.messages)
    }
  }
}
</script>

