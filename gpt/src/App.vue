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
      if (this.selectedPreset.name === "Chat") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: this.prompt }
        }
      } else if (this.selectedPreset.name === "Outline Essay") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Create an outline for an essay about: " + this.prompt }
        }
      } else if (this.selectedPreset.name === "Sarcastic") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Reply to this in a sarcastic manner: " + this.prompt }
        }
      } else if (this.selectedPreset.name === "Write Essay") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Write an essay about: " + this.prompt }
        }
      } else if (this.selectedPreset.name === "Explain Code") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Explain this code: " +  this.prompt } 
        }
      } else if (this.selectedPreset.name === "Rewrite") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Rewrite this: " + this.prompt }
        }
      } else if (this.selectedPreset.name === "Shakespear Rewrite") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Rewrite this as if shakespear wrote it: " + this.prompt }
        }
      } else if (this.selectedPreset.name == "Angry chat") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Reply in a rude manner using curse words: " + this.prompt }
        }
      } else {
        console.log("no matching preset")
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
          "max_tokens": 2000,
          "top_p": 1,
          "frequency_penalty": 0,
          "presence_penalty": 0.6,
          "stop": [" Human:", " AI:"]
        }
      },
      {
        name: "Outline Essay",
        url: 'https://api.openai.com/v1/completions',
        start: "Create an outline for an essay about: ",
        body: {
          "model": "text-davinci-003",
          "prompt": "Create an outline for an essay about: " + this.prompt,
          "max_tokens": 2000,
          "stop": ["\"\"\""]
        }
      },
      {
        name: "Write Essay",
        url: 'https://api.openai.com/v1/completions',
        start: "Write an essay about: ",
        body: {
          "model": "text-davinci-003",
          "prompt": "Write an essay about: " + this.prompt,
          "max_tokens": 2000,
          "stop": ["\"\"\""]
        }
      },
      {
        name: "Rewrite",
        url: 'https://api.openai.com/v1/completions',
        start: "Rewrite this: ",
        body: {
          "model": "text-davinci-003",
          "prompt": "Rewrite this: " + this.prompt,
          "max_tokens": 2000,
          "stop": ["\"\"\""]
        }
      },
      {
        name: "Shakespear Rewrite",
        url: 'https://api.openai.com/v1/completions',
        start: "Rewrite this as if shakespear wrote it: ",
        body: {
          "model": "text-davinci-003",
          "prompt": "Rewrite this as if shakespear wrote it: ",
          "max_tokens": 2000,
          "stop": ["\"\"\""]
        }
      },
      {
        name: "Explain Code",
        url: 'https://api.openai.com/v1/completions',
        body: {
          "model": "text-davinci-003",
          "prompt": "Explain this code: " + this.prompt,
          "max_tokens": 2000,
          "stop": ["\"\"\""]
        }
      },
      {
        name: "Sarcastic",
        url: 'https://api.openai.com/v1/completions',
        start: "Reply to this in a sarcastic manner: ",
        body: {
          "model": "text-davinci-003",
          "prompt": "Reply to this in a sarcastic manner: " + this.prompt,
          "max_tokens": 2000,
          "stop": ["\"\"\""]
        }
      },
      {
        name: "Angry chat",
        url: 'https://api.openai.com/v1/completions',
        body: {
          "model": "text-davinci-003",
          "prompt": this.prompt,
          "temperature": 0,
          "max_tokens": 2000,
          "top_p": 1,
          "frequency_penalty": 0,
          "presence_penalty": 0.6,
          "stop": [" Human:", " AI:"]
        }
      }
    ]
    this.selectedPreset = this.presets[0]
  },
  methods: {
    async getResponse() {
      this.messages.push({ id: Date.now(), sender: "You", text: this.prompt })
      try {
        console.log(this.currentPreset.url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer YOUR_API_KEY`
          },
          body: JSON.stringify(this.currentPreset.body)
        })
        const response = await fetch(this.currentPreset.url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer YOUR_API_KEY`
          },
          body: JSON.stringify(this.currentPreset.body)
        });
        this.prompt = "";
        const json = await response.json();
        this.messages.push({ id: Date.now(), sender: "AI", text: json.choices[0].text });
      } catch (error) {
        console.error(error);
      }
    }
  }
}

</script>