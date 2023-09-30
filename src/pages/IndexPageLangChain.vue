<template>
  <q-page class="padding">
    <div class="main">
      <q-form class="form" @submit="generateResponse">
        <q-input filled label="Type a question" color="info" v-model="input" style="width: 95%" />
        <q-btn label="Submit" type="submit" color="primary" @click="submit"></q-btn>
      </q-form>

      <b-row>
        <q-btn label="Convert Data" color="primary" @click="storeData()" style="width: 100%" class="mt-5"></q-btn>
      </b-row>

      <!-- <q-file color="lime-11" bg-color="green" filled v-model="fileModel" label="Select File">
        <template v-slot:prepend>
          <q-icon name="attachment" />
        </template>
      </q-file> -->

      <q-file filled bottom-slots v-model="fileModel" class="mt-5" label="Select a file to use for LLM" counter>
        <template v-slot:prepend>
          <q-icon name="cloud_upload" @click.stop.prevent />
        </template>
        <template v-slot:append>
          <q-icon name="close" @click.stop.prevent="model = null" class="cursor-pointer" />
        </template>

        <template v-slot:hint> Field hint </template>
      </q-file>

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
import { defineComponent } from "vue";
import { LLMChain } from "langchain/chains";
import { OpenAI } from "langchain/llms/openai";
import { PromptTemplate } from "langchain/prompts";
import { TextLoader } from "langchain/document_loaders/fs/text";
import { CharacterTextSplitter } from "langchain/text_splitter";
import { DirectoryLoader } from "langchain/document_loaders/fs/directory";

export default defineComponent({
  name: "IndexPage",
  data() {
    return {
      //   OPEN_API_KEY: "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", //process.env.openAIAPI ,
      input: "",
      response: "",
      log: [],
      messages: [],
      fileModel: "",
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
    async storeData() {

      // Fixed
      const directoryLoader = new DirectoryLoader('src/data', {
        '.txt': (path) => { console.log(path); new TextLoader(path) }
      });

      const docs = await directoryLoader.load();

      const splitter = new CharacterTextSplitter({
        chunkSize: 200,
        chunkOverlap: 50,
      });

      const documents = await splitter.splitDocuments(docs);
      console.log(documents);
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
