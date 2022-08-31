<template>
  <pre class="whitespace-pre-wrap p-2">
    <form class="w-full max-w-sm">
        <div class="md:flex md:items-center mb-2">
            <div class="md:w-1/2">
                <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4 pb-10" for="inline-full-name">
                Temperature 
                </label>
            </div>
            <div class="md:w-1/2 flex flex-col">
                <input v-model="temperature" class="text-right bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="inline-full-name" type="text">
                <input v-model="temperature" type="range" min="0" max="1" step="0.01"/>
            </div>
        </div>
        <div class="md:flex md:items-center mb-2">
            <div class="md:w-1/2">
                <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4 pb-10">
                Maximum Length 
                </label>
            </div>
            <div class="md:w-1/2 flex flex-col">
                <input v-model="maxLength" class="text-right bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="inline-full-name" type="text">
                <input v-model="maxLength" type="range" min="1" max="4000" step="1"/>
            </div>
        </div>
    </form>
  </pre>
<!-- {{ JSON.stringify(markdownBlocks, null, 2) }} -->
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const temperature = ref(0.7)
const maxLength = ref(256)

defineExpose({ temperature, maxLength })

</script>
