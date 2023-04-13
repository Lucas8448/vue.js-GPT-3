<template>
  <div class="title">
    <h1>ChatBot<font size=1>Powered by OpenAI</font></h1>
  </div>
  <div class="container">
    <div class="options">
      <div class="options-title">
        <h1>Options</h1>
      </div>
      <div class="clear-chat">
        <button @click="clearChat">Clear Chat</button>
      </div>
    </div>
    <div class="chat-container" ref="chatContainer">
      <div class="messages" v-for="message in messages" :key="message.id" >
        <div class="message" :class="{ 'You': message.sender === 'You', 'AI': message.sender === 'AI' }">
          {{ message.sender }} : {{ message.text }}
        </div>
      </div>
    </div>
    <div class="inputs">
      <div class="input-container">
        <input type="text" v-model="prompt" @keyup.enter="getResponse" placeholder="Enter your message here"
          class="input-area">
      </div>
      <div class="input-container">
          <input type="text" v-model="apiKey" placeholder="Paste your OpenAI API key here"
            class="input-area">
        </div>
      <select class="chatbot-dropdown" v-model="selectedPreset">
        <option v-for="preset in presets" :value="preset">{{ preset.name }}</option>
      </select>
    </div>
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
      previousChats: null,
      apiKey: null,
    }
  },
  updated() {
    this.$nextTick(() => {
      this.$refs.chatContainer.scrollTop = this.$refs.chatContainer.scrollHeight
    })
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
      } else if (this.selectedPreset.name === "Shakespeare Rewrite") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Rewrite this as if Shakespeare wrote it: " + this.prompt }
        }
      } else if (this.selectedPreset.name == "Angry chat") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: { ...this.selectedPreset.body, prompt: "Reply in a rude manner using curse words: " + this.prompt }
        }
      } else if (this.selectedPreset.name == "Stable Diffusion Prompt") {
        return {
          name: this.selectedPreset.name,
          url: this.selectedPreset.url,
          body: {
            ...this.selectedPreset.body, prompt: "Create a new unique prompt with the same format as these ones: 'portrait photo of a asia old warrior chief, tribal panther make up, blue on red, side profile, looking away, serious eyes, 50mm portrait photography, hard rim lighting. or city made out of glass : : close shot : : 3 5 mm, realism, octane render, 8 k, exploration, cinematic, trending on artstation, realistic, 3 5 mm camera, unreal engine, hyper detailed, photo – realistic maximum detail, volumetric light, moody cinematic epic concept art, realistic matte painting, hyper photorealistic, concept art, volumetric light, cinematic epic, octane render, 8 k, corona render, movie concept art, octane render, 8 k, corona render, cinematic, trending on artstation, movie concept art, cinematic composition, ultra – detailed, realistic, hyper – realistic, volumetric lighting, 8 k. or A type specimen poster showing every letter in the alphabet where each letter is made of pieces of simple shapes like circles, squares, triangles, and diamonds like color-forms very colorful and vibrant::2 poster in the international typographic style showing line-art illustrations of a full color spectrum typeface graphic design —h 432 —vibe. Using this initial promt (no repeating keywords): " + this.prompt }
        }
      } else {
        console.log("no matching preset")
      }
    }
  },
  mounted() {
    window.addEventListener('beforeunload', (event) => {
      localStorage.setItem('previousChats', JSON.stringify(this.messages))
    });
    this.previousChats = JSON.parse(localStorage.getItem('previousChats')) || []
    console.log("Loaded chats: " + this.previousChats)
    this.messages = [...this.previousChats]
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
        name: "Shakespeare Rewrite",
        url: 'https://api.openai.com/v1/completions',
        start: "Rewrite this as if Shakespeare wrote it: ",
        body: {
          "model": "text-davinci-003",
          "prompt": "Rewrite this as if Shakespeare wrote it: ",
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
      },
      {
        name: "Stable Diffusion Prompt",
        url: 'https://api.openai.com/v1/completions',
        body: {
          "model": "text-davinci-003",
          "prompt": "Create a Positive and Negative prompt based on three key words (medium, motive, description/details), where the positive prompt uses 1/6 as meduim, 2/6 as motive and 2-3/6 as description/details" +  this.prompt,
          "temperature": 0,
          "max_tokens": 150,
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
            'Authorization': f`Bearer ${this.apiKey}`
          },
          body: JSON.stringify(this.currentPreset.body)
        });
        this.prompt = "";
        const json = await response.json();
        console.log(json);
        this.messages.push({ id: Date.now(), sender: "AI", text: json.choices[0].text });
      } catch (error) {
        console.error(error);
      }
    },
    async clearChat() {
      this.messages = []
      localStorage.setItem('previousChats', JSON.stringify(this.messages))
    }
  }
}

</script>