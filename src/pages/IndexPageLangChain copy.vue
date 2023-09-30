<template>
  <q-page class="padding">
    <div class="main">
      <q-form class="form" @submit="generateResponse">
        <q-input filled label="Type a question" color="info" v-model="input" style="width: 95%" />
        <q-btn label="Submit" type="submit" color="primary" @click="submit"></q-btn>
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
//import { CSVLoader } from "langchain/document_loaders/fs/csv";
//import { CSVLoader } from "langchain/document_loaders";
//import { Document } from "langchain/document";
//import { OpenAI } from "langchain/llms/openai";
//import { OpenAI } from "langchain/llms";
//import { OpenAIEmbeddings } from "langchain/embeddings/openai";
//import {VectorstoreIndexCreator} from "langchain/indexes"
//import { ChatOpenAI } from "langchain/chat_models/openai";

import { defineComponent } from "vue";
//import {Configuration, OpenAIApi} from 'openai';
//import OpenAI from 'openai';
import { OpenAI } from "langchain/llms/openai";
import { PromptTemplate } from "langchain/prompts";
import { LLMChain } from "langchain/chains";
import { StructuredOutputParser } from "langchain/output_parsers";

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
      const model = new OpenAI({ temperature: 0 });
      const template = "Be professional\n Question: {question}";
      const prompt = new PromptTemplate({
        template,
        inputVariables: ["question"],
      });
      const chain = new LLMChain({ llm: model, prompt });
      const result = await chain.call({ question: input });
      //  this.messages.push({role: chatCompletion.choices[0].message.role, content: chatCompletion.choices[0].message.content});
      //  this.response = chatCompletion.choices[0].message.content;

      this.response = result.text;
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

      // const model = new OpenAI({ temperature: 0 });
      //const template = "Be very funny when answering questions\n Question: {question}";
      //const prompt = new PromptTemplate({ template, inputVariables: ["question"] });

      //const chain = new LLMChain({ llm: model, prompt });

      //const result = await chain.call({ question: this.input });
      //console.log(result);
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
