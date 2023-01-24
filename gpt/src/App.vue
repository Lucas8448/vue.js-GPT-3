<template>
  <div class="chat-container">
    <div class="messages" v-for="message in messages" :key="message.id">
      <div class="message" :class="{ 'user': message.sender === 'user', 'chatbot': message.sender === 'chatbot' }">
        {{ message.sender }} : {{ message.text }}
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
      presets: [],
    }
  },
  computed: {
    currentPreset: function () {
      return {
        name: this.selectedPreset.name,
        url: this.selectedPreset.url,
        start: this.selectedPreset.start,
        body: { ...this.selectedPreset.body, prompt: this.selectedPreset.start + ' ' + this.prompt }
      }
    }
  },
  mounted() {
    this.presets = [
      {
        name: "Chat",
        url: 'https://api.openai.com/v1/completions',
        body: {
          "model": "text-davinci-003",
          "prompt": this.prompt,
          "temperature": 0,
          "max_tokens": 150,
          "top_p": 1,
          "frequency_penalty": 0,
          "presence_penalty": 0.6,
          "stop": [" Human:", " AI:"]
        }
      },
      {
        name: "Outline Essay",
        url: 'https://api.openai.com/v1/engines/davinci/completions',
        start: "Create an outline for an essay about: ",
        body:{
          "engine": "davinci",
          "prompt": "Create an outline for an essay about: " + this.prompt,
          "stop": ["\"\"\""]
        }
      },
      {
        name: "Explain Code",
        url: 'https://api.openai.com/v1/engines/code-davinci/completions',
        body: {
          "engine": "code-davinci",
          "prompt": this.prompt,
          "stop": ["\"\"\""]
        }
      },
      {
        name: "Sarcasm",
        url: 'https://api.openai.com/v1/engines/davinci/completions',
        body: {
          "engine": "davinci",
          "prompt": this.prompt,
          "temperature": 0.9,
          "max_tokens": 1000,
          "stop": ["\"\"\""]
        }
      }
    ]
    this.selectedPreset = this.presets[0]
  },
  methods: {
    async getResponse() {
      this.messages.push({ id: Date.now(), sender: "user", text: this.prompt })
      try {
        console.log(this.currentPreset.url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer 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`
          },
          body: JSON.stringify(this.currentPreset.body)
        })
        const response = await fetch(this.currentPreset.url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer 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`
          },
          body: JSON.stringify(this.currentPreset.body)
        });
        this.prompt = "";
        const json = await response.json();
        this.messages.push({ id: Date.now(), sender: "chatbot", text: json.choices[0].text });
      } catch (error) {
        console.error(error);
      }
    }
  }
}

</script>