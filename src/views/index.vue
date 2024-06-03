<template>
  <el-drawer
      v-model="drawer"
      :modal="true"
      :close-on-click-modal="true"
      size="25%"
  >
    <template #header>
      <span class="ai-title">智网优评——智能AI助手</span>
    </template>
    <template #default>
      <div class="container">
        <div class="msg" ref="listEl">
          <Msg v-for="(msg, index) in msgList" :role="msg.role" :x="left" :y="top" :content="msg.content"
               :key="index"></Msg>
          <Msg v-if="streaming" role="system" :content="streamingText" :streaming="true"></Msg>
        </div>
      </div>
    </template>
    <template #footer>
      <div class="input">
        <InputDiv v-model:value="inputValue" @submit="handleSubmit"></InputDiv>
      </div>
    </template>
  </el-drawer>

  <draggable-container :disabled="true">
    <vue3-draggable-resizable
        :initW="w"
        :initH="h"
        v-model:x="top"
        v-model:y="left"
        v-model:w="w"
        v-model:h="h"
        v-model:active="active"
        :parent="true"
        :draggable="true"
        :resizable="false"
        class="ai-container"
        @dblclick="drawer = true"
    >
      <img src="../assets/ai_assistant.png">
    </vue3-draggable-resizable>
  </draggable-container>

</template>
<script lang="ts" setup>
import {ref, nextTick, watch} from 'vue'
import InputDiv from '../components/InputDiv.vue'
import Msg from '../components/Msg.vue'
import {useScroll} from '@vueuse/core'
import {useGpt} from '../hooks/useGpt'
import config from '../config'
import Vue3DraggableResizable from "vue3-draggable-resizable";
import {DraggableContainer} from 'vue3-draggable-resizable'

interface Msg {
  role: string
  content: string
}

const drawer = ref(false)

const top = ref(0)
const left = ref(0)
const w = ref(60)
const h = ref(60)
const active = ref(true)

const inputValue = ref('')
const {msgList, streaming, streamingText, stream} = useGpt(config.key, true)
const listEl = ref()
const {y} = useScroll(listEl)
const scrollToBottom = () => {
  nextTick(() => {
    y.value = listEl.value?.scrollHeight || 0
  })
}
watch(streamingText, (val) => {
  if (val) {
    scrollToBottom()
  }
})
const handleSubmit = (content: string) => {
  if (content === '') return
  stream(content)
}
</script>
<style lang="scss" scoped>
.input {
  width: 100%;
  min-height: 6vh;
  overflow-y: auto;
  text-align: left;
}

.ai-title {
  font-size: 24px;
  color: #0087ff;
  font-weight: bold;
  text-align: center;
}

.container {
  height: 100%;
  width: 100%;
  display: flex;
  flex-direction: column;
  background-color: #f5f5f5;

  .msg {
    width: 100%;
    flex: 1;
    overflow-y: auto;
    padding: 24px;
  }

}

.ai-container {
  //background-color: #629eff;
  border-radius: 50%;
  display: flex;
  justify-content: center;
}

</style>