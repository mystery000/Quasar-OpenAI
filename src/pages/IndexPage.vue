<template>
  <q-page class="padding">
    <div class="main">
      <q-form class="form" @submit="generateResponse">
        <q-input filled label="Type a question" color="info" v-model="input" style="width: 95%" />
        <q-btn label="Submit" type="submit" color="primary" @keydown.enter.prevent="submit" @click="submit"></q-btn>
      </q-form>
      <div class="chatLog">
        <q-card v-for="(item, index) in log" :key="index">
          <q-card-section>
            <div class="text-h6">User: {{ item.input }}</div>
            <div class="text-h6">Assistant: {{ item.response }}</div>
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script>
import dotenv from "dotenv";

import { defineComponent } from "vue";

import OpenAI from "openai";

export default defineComponent({
  name: "IndexPage",
  data() {
    return {
      //   OPEN_API_KEY: "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", //process.env.openAIAPI ,
      input: "",
      response: "",
      log: [],
      messages: [],
    };
  },
  computed: {
    api_key() {
      // console.log (process.env.OPENAI_API_KEY)
      return process.env.OPENAI_API_KEY;
    },
  },
  methods: {
    async completionCall(input) {
      //  this.messages.push({role: "user", content: input});

      // const configuation = new Configuration({
      //   apiKey: process.env.OPEN_API_KEY || this.OPEN_API_KEY,
      // });
      // const openai = new OpenAI({
      //   apiKey: process.env.OPENAI_API_KEY || this.OPEN_API_KEY, // This is also the default, can be omitted
      //    });

      const openai = new OpenAI({
        apiKey: process.env.OPENAI_API_KEY,
        dangerouslyAllowBrowser: true,
        // apiKey: process.env.OPENAI_API_KEY
        //piKey: this.OPEN_API_KEY,  dangerouslyAllowBrowser: true // This is also the default, can be omitted
      });

      const chatCompletion = await openai.chat.completions.create({
        model: "gpt-3.5-turbo",
        messages: [{ role: "user", content: this.input }],
        temperature: 0,
      });

      this.messages.push({
        role: chatCompletion.choices[0].message.role,
        content: chatCompletion.choices[0].message.content,
      });
      this.response = chatCompletion.choices[0].message.content;
      console.log(chatCompletion.choices[0].message);
    },
    async generateResponse() {
      await this.completionCall(this.input).then(() => {
        this.log.unshift({
          input: this.input,
          response: this.response,
        });
        this.input = "";
        this.response = "";
      });
      console.log(this.input);
    },
  },
});
</script>
<style>
.main {
  max-width: 1200px;
  margin: 0 auto;
}

.form {
  display: flex;
  width: 100%;
}
</style>
