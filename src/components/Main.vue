<template>
  <div class="flex">
    <div class="sticky top-0 h-screen overflow-y-auto bg-neutral-50">
      <!-- <Markdown :page="page" /> -->
    </div>
    <div class="shrink-0 px-24 min-w-[50%] mx-auto">
      <Lotion :page="page" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { BlockType } from '@/utils/types'
import Markdown from './Markdown.vue'
import Lotion from './Lotion.vue'
import { v4 as uuidv4 } from 'uuid';

const page = ref({
  name: 'HyperPrompt',
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
