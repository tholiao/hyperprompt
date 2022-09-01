<template>
  <div class="flex">
    <div class="shrink-0 px-24 min-w-[50%] mx-auto">
      <Lotion :page="page" />
    </div>
    <div class="sticky top-0 h-screen bg-neutral-50 max-w-sm flex-col flex">
      <form class="w-full h-full max-w-sm align-middle space-y-5">
          <div class="md:flex md:items-center mb-2">
              <div class="md:w-1/2">
                  <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4">Temperature </label>
              </div>
              <div class="md:w-1/2 flex flex-col">
                  <input v-model.number="page.temperature" class="text-right bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="inline-full-name" type="number">
                  <input v-model.number="page.temperature" type="range" min="0" max="1" step="0.01"/>
              </div>
          </div>
          <div class="md:flex md:items-center mb-2">
              <div class="md:w-1/2">
                  <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4 pb-10">
                  Maximum Length 
                  </label>
              </div>
              <div class="md:w-1/2 flex flex-col">
                  <input v-model.number="page.maxLength" class="text-right bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="inline-full-name" type="number">
                  <input v-model.number="page.maxLength" type="range" min="1" max="4000" step="1"/>
              </div>
          </div>
          <div class="md:flex md:items-center mb-2">
              <div class="md:w-1/2">
                  <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4 pb-10">Top P</label>
              </div>
              <div class="md:w-1/2 flex flex-col">
                  <input v-model.number="page.topP" class="text-right bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="inline-full-name" type="number">
                  <input v-model.number="page.topP" type="range" min="0" max="1" step="0.01"/>
              </div>
          </div>
          <div class="md:flex md:items-center mb-2">
              <div class="md:w-1/2">
                  <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4 pb-10">
                  Frequency Penalty
                  </label>
              </div>
              <div class="md:w-1/2 flex flex-col">
                  <input v-model.number="page.frequencyPenalty" class="text-right bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="inline-full-name" type="number">
                  <input v-model.number="page.frequencyPenalty" type="range" min="-2" max="2" step="0.01"/>
              </div>
          </div>
          <div class="md:flex md:items-center mb-2">
              <div class="md:w-1/2">
                  <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4 pb-10">
                  Presence Penalty
                  </label>
              </div>
              <div class="md:w-1/2 flex flex-col">
                  <input v-model.number="page.presencePenalty" class="text-right bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="inline-full-name" type="number">
                  <input v-model.number="page.presencePenalty" type="range" min="-2" max="2" step="0.01"/>
              </div>
          </div>
      </form>
    </div>
  </div>

</template>

<script setup lang="ts">
import { ref } from 'vue'
import { BlockType } from '@/utils/types'
import Lotion from './Lotion.vue'
import { v4 as uuidv4 } from 'uuid';

const page = ref({
  name: 'HyperPrompt',
  temperature: 0.7,
  maxLength: 256,
  topP: 1,
  frequencyPenalty: 0,
  presencePenalty: 0,
  showSpinner: true,
  showCopied: false,
  blocks:[
  {
    id: uuidv4(),
    type: BlockType.Text,
    details: {
      value: '<p style="color:green">👋 Welcome! This is a private page for you to play around with.</p>'
    },
  },
  {
    id: uuidv4(),
    type: BlockType.Text,
    details: {
      value: '📥 Each line is a block. Text is passed in the order of the blocks as input to GPT-3.'
    },
  },
  {
    id: uuidv4(),
    type: BlockType.Text,
    details: {
      value: '📤 This will generate an output, which is added as another block.'
    },
  },
  {
    id: uuidv4(),
    type: BlockType.Text,
    details: {
      value: '💾 Everything above the divider is in the buffer and is not sent to the model. You can store work in progress prompts here.'
    },
  },
  {
    id: uuidv4(),
    type: BlockType.Divider,
    details: {
      value: 'Divider'
    },
  },
  {
    id: uuidv4(),
    type: BlockType.Text,
    details: {
      value: 'Everything below the divider is sent to the model.'
    },
  },
  {
    id: uuidv4(),
    type: BlockType.Text,
    details: {
      value: 'Delete these blocks to get started.'
    },
  },
  {
    id: uuidv4(),
    type: BlockType.Text,
    details: {
      value: 'Who was Marcus Aurelius?'
    },
  },

  ]
})
</script>
