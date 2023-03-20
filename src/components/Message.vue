<script setup lang="ts">
import {computed} from 'vue';
import {parse} from 'marked';
const props = defineProps({
    text: {type:String,required:true},
    role: {type:String,required:true}
})

const bgColour = props.role==="user"?'#494':'#eef'
const marginLeft = "6px 6px 6px 50px";
const marginRight = "6px 50px 6px 6px";
const margin = props.role==="user"?marginLeft:marginRight;
const markdownToHTML = computed(()=>parse(props.text))
</script>

<template>
    <div class="bubble" v-html="markdownToHTML" />
</template>

<style>
.bubble {
    border-radius: 6px;
    padding: 20px;
    background-color: v-bind(bgColour);
    width: 90%;
    margin: v-bind(margin);
}
</style>