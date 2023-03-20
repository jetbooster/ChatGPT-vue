<script setup lang="ts">
import Message from './Message.vue'
import {Configuration,OpenAIApi, type ChatCompletionRequestMessage,ChatCompletionRequestMessageRoleEnum, ChatCompletionResponseMessageRoleEnum} from 'openai';
import { ref, computed, type Ref } from 'vue';
const conf = new Configuration({
  apiKey: import.meta.env.VITE_OPENAI_API_KEY
});

const openai = new OpenAIApi(conf);

const wait = (ms:number) => new Promise((res)=>{setTimeout(res,ms)})

const text = ref("blah");
const messages: Ref<ChatCompletionRequestMessage[]>= ref([])

const updatedMessages = computed({ 
  get () {
    return [...messages.value]
  },
  set (val) {
    messages.value = val;
  }
})

const submit = async () =>{
  if (text.value === ""){
    return
  }
  const message:ChatCompletionRequestMessage = {
    role: ChatCompletionResponseMessageRoleEnum.User,
    content: text.value
  }
  console.log(message);
  updatedMessages.value = [...updatedMessages.value,message]
  console.log(messages);
  text.value = "";
  console.log('pre-promise');
  const promise = openai.createChatCompletion({
    model: 'gpt-3.5-turbo',
    messages:[
      {role:"system",content: 'You are a snarky assistant that responds in the style of a reddit comment.'},
      ...updatedMessages.value
    ]
  })
  // const promise = wait(2000);
  console.log('post-promise');
  updatedMessages.value = [...updatedMessages.value,{role:'assistant',content:"..."}];
  const {data} = await promise;
  const response = data.choices[0]
  console.log('post-resolved');
  // const response = {finish_reason: 'stop', message: {role:ChatCompletionRequestMessageRoleEnum.Assistant,content: 'completed!'}};
  if (response.finish_reason === 'stop' && response.message){
    updatedMessages.value.pop();
    updatedMessages.value = [...updatedMessages.value,response.message];
    console.log(messages)
  }
}

</script>

<template>
  <div class="chatarea">
    <div class="messagewindow">
      <Message v-if="updatedMessages.length===0" text="Hello! Type your message to ChatGPT" role="Assistant"/>
      <Message v-for="(message,i) in updatedMessages" :text="message.content" :key="i" :role="message.role"/>
      <div id="anchor"></div>
    </div>
    <form class="banner" @submit.prevent="submit">
      <input class="entry" v-model="text" type="text" label="test" placeholder="Enter request to chatgpt" cols="40"/>
      <button class="button" :disabled="!text">Submit</button>
    </form>
  </div>
</template>

<style scoped>

.chatarea {
  border: 1px solid black;
  background-color: #eee;
}

.messagewindow {
  height: 400px;
  overflow-y: auto;
}

.messagewindow * {
  overflow-anchor: none;
}

#anchor {
  overflow-anchor: auto;
  height: 1px;
}
.banner {
  background-color: gray;
  padding: 2px;
  display: flex
}
.entry {
  border: 1px solid black;
  border-radius: 4px;
  padding: 0.2em 0.6em;
  flex:20
}
.button {
  border-radius: 4px;
  border-style: none;
  margin: 2px;
  flex: 1;
}
</style>
