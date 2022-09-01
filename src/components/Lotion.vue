<template>
  <div class="w-[65ch] mx-auto my-24" ref="editor">
    <h1 id="title" contenteditable="true" spellcheck="false" data-ph="Untitled" @blur="props.page.name=($event.target as HTMLElement).innerText.replace('\n', '')"
      class="px-4 sm:px-0 focus:outline-none focus-visible:outline-none text-5xl font-bold mb-12"
      :class="props.page.name ? '' : 'empty'">
      {{ props.page.name || '' }}
    </h1>    
    <p class="pb-10">OpenAI API Key: <input v-model="key" placeholder="Paste Here"/></p>

    <!-- <div class="border-2 rounded-md border-black pl-20 p-2"> -->
      <draggable id="blocks" tag="div" :list="props.page.blocks"  handle=".handle"
        v-bind="dragOptions" class="-ml-24 space-y-2 pb-4">
        <transition-group type="transition">
          <BlockComponent :block="block" v-for="block, i in props.page.blocks" :key="i" :id="'block-'+block.id"
            :ref="el => blockElements[i] = (el as unknown as typeof Block)"
            @deleteBlock="deleteBlock(i)"
            @newBlock="insertBlock(i, '')"
            @moveToPrevChar="() => { if (blockElements[i-1]) blockElements[i-1].moveToEnd(); scrollIntoView(); }"
            @moveToNextChar="() => { if (blockElements[i+1]) blockElements[i+1].moveToStart(); scrollIntoView(); }"
            @moveToPrevLine="() => { if (blockElements[i-1]) blockElements[i-1].moveToLastLine(); scrollIntoView(); }"
            @moveToNextLine="() => { if (blockElements[i+1]) blockElements[i+1].moveToFirstLine(); scrollIntoView(); }"
            @merge="merge(i)"
            @split="split(i)"
            @setBlockType="type => setBlockType(i, type)"
            />
        </transition-group>
      </draggable>
    <!-- </div> -->
    <!-- </div> -->

    <div class="flex flex-row align-middle content-center">
      <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold px-4 py-2 rounded">
        Submit
      </button>
      <svg v-if="props.page.showSpinner" aria-hidden="true" class="mr-2 ml-2 mt-1 w-8 h-8 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
            <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
      </svg>
    </div>
  </div>
</template>

<script setup lang="ts">
import { Configuration, OpenAIApi } from 'openai'
import { ref, onBeforeUpdate, PropType } from 'vue'
import { VueDraggableNext as draggable } from 'vue-draggable-next'
import { Block, BlockType, isTextBlock } from '@/utils/types'
// import { temperature, maxLength } from './Settings.vue'
import BlockComponent from './Block.vue'
import { v4 as uuidv4 } from 'uuid';

const key = ref('')

const props = defineProps({
  page: {
    type: Object as PropType<{ 
      name:string, 
      temperature:number, 
      maxLength:number, 
      topP:number,
      frequencyPenalty:number,
      presencePenalty:number,
      blocks:Block[],
      showSpinner: boolean
    }>,
    required: true,
  }
})


const editor = ref<HTMLDivElement|null>(null)
document.addEventListener('mousedown', (event:MouseEvent) => {
  console.log(typeof(props.page.temperature), typeof(props.page.maxLength), props.page.blocks)
  const input = event.target as HTMLElement;
  if (input.innerText == "Submit") {
    props.page.showSpinner = true
    console.log(key.value)
    const configuration = new Configuration({
        apiKey: key.value,
    });
    const openai = new OpenAIApi(configuration);

    let inputArray = []
    let postDivider = false
    for (let i = 0; i < props.page.blocks.length; i++) {
      postDivider = props.page.blocks[i].type == "DIVIDER" || postDivider
      if (blockElements.value[i] && postDivider) {
        inputArray.push(blockElements.value[i].getTextContent())
      }
    }
    const inputString = inputArray.join('\n')
    console.log(inputString)
    openai.createCompletion({
      model: "text-davinci-002",
      prompt: inputString,
      max_tokens: props.page.maxLength,
      temperature: props.page.temperature,
      top_p: props.page.topP,
      frequency_penalty: props.page.frequencyPenalty,
      presence_penalty: props.page.presencePenalty,
    }).then((response) => {
      props.page.showSpinner = false
      // blockElements.value[blockElements.value.length-1].value = response.data.choices[0].text
      insertBlock(props.page.blocks.length-1, response.data.choices[0].text)
      
    });
  } else {
    // Automatically focus on nearest block on click
    const blocks = document.getElementById('blocks')
    const title = document.getElementById('title')
    const editorRect = editor.value?.getClientRects()[0]
    // Check that click is outside Editor
    if ((event.clientX < ((editorRect as DOMRect).left || -1)) || (event.clientX > (editorRect?.right || window.innerWidth))) {
      // Focus on title
      const titleRect = title?.getClientRects()[0]
      if (event.clientY > (titleRect?.top || window.innerHeight) && event.clientY < (titleRect?.bottom || 0)) {
        // Check if click is on left or right side
        const rect = title?.getClientRects()[0]
        let moveToStart = true
        if (event.x > (rect as DOMRect).right) moveToStart = false 
        const selection = window.getSelection()
        const range = document.createRange()
        range.selectNodeContents(title as Node)
        range.collapse(moveToStart)
        selection?.removeAllRanges()
        selection?.addRange(range)
        return
      }
      // or nearest block
      const blockRects = Array.from(blocks?.children as HTMLCollection)
      const block = blockRects.find(child => {
        const rect = child.getClientRects()[0]
        return event.clientY > rect.top && event.clientY < rect.bottom
      })
      const blockIdx = blockRects.findIndex(child => {
        const rect = child.getClientRects()[0]
        return event.clientY > rect.top && event.clientY < rect.bottom
      })
      if (block) {
        // Check if click is on left or right side
        const rect = block.getClientRects()[0]
        if (event.x < rect.left) {
          // Move to start of block
          blockElements.value[blockIdx].moveToStart()
        } else {
          // Move to end of block
          blockElements.value[blockIdx].moveToEnd()
        }
        return
      }
    }
    // If cursor is between Submit button and last block, insert block there 
    const lastBlockRect = blocks?.lastElementChild?.getClientRects()[0]
    if (!lastBlockRect) return
    if (event.clientX > (lastBlockRect as DOMRect).left && event.clientX < (lastBlockRect as DOMRect).right
      && event.clientY > (lastBlockRect as DOMRect).bottom) {
        const lastBlock = props.page.blocks[props.page.blocks.length-1]
        const lastBlockComponent = blockElements.value[props.page.blocks.length-1]
        if (lastBlock.type === BlockType.Text && lastBlockComponent.getTextContent() === '') {
          // If last block is empty Text, focus on last block
          setTimeout(lastBlockComponent.moveToEnd)
        } else {
          // Otherwise add new empty Text block
          insertBlock(props.page.blocks.length-1, '')
          // setBlockType(props.page.blocks.length-1, BlockType.Text)
        }
      }
  }
})

const dragOptions = {
  animation: 150,
  group: 'blocks',
  disabled: false,
  ghostClass: 'ghost',
}

onBeforeUpdate(() => {
  blockElements.value = []
})

const blockElements = ref<typeof BlockComponent[]>([])

function scrollIntoView () {
  const selection = window.getSelection()
  if (selection?.anchorNode?.nodeType === Node.ELEMENT_NODE) {
    (selection?.anchorNode as HTMLElement).scrollIntoView({behavior: "smooth", block: "center", inline: "nearest"})
  } else {
    (selection?.anchorNode?.parentElement as HTMLElement).scrollIntoView({behavior: "smooth", block: "center", inline: "nearest"})
  }
}

function insertBlock (blockIdx: number, value: string) {
  if (value) {
    props.page.blocks.splice(blockIdx + 1, 0, {
      id: uuidv4(),
      type: BlockType.Text,
      details: {
        // https://tiptap.dev/api/extensions/color for color
        value: '<p><strong>' + value + '</strong></p>',
      },
    })
  } else {
    props.page.blocks.splice(blockIdx + 1, 0, {
      id: uuidv4(),
      type: BlockType.Text,
      details: {
        value: '<p></p>',
      },
    })

  }
  setTimeout(() => {
    blockElements.value[blockIdx+1].moveToStart()
    scrollIntoView()
  })
}

function deleteBlock (blockIdx: number) {
  props.page.blocks.splice(blockIdx, 1)
  // Always keep at least one block
  if (props.page.blocks.length === 0) {
    insertBlock(0, '')
  }
}

function setBlockType (blockIdx: number, type: BlockType) {
  props.page.blocks[blockIdx].details.value = blockElements.value[blockIdx].getTextContent()
  props.page.blocks[blockIdx].type = type
  if (type === BlockType.Divider) {
    props.page.blocks[blockIdx].details = {}
    insertBlock(blockIdx, '')
  } else setTimeout(() => blockElements.value[blockIdx].moveToEnd())
}

function merge (blockIdx: number) {
  // When deleting the first character of non-text block
  // the block should first turn into a text block
  if([BlockType.H1, BlockType.H2, BlockType.H3,BlockType.Quote]
      .includes(props.page.blocks[blockIdx].type)){
    setBlockType(blockIdx, BlockType.Text)
    setTimeout(()=>{
      blockElements.value[blockIdx].moveToStart()
    })
    return
  }

  if (blockIdx === 0) return

  if (isTextBlock(props.page.blocks[blockIdx-1].type)) {
    const prevBlockContentLength = blockElements.value[blockIdx-1].getTextContent().length
    props.page.blocks[blockIdx-1].details.value = ('<p>' + (props.page.blocks[blockIdx-1] as any).details.value.replace('<p>', '').replace('</p>', '') + blockElements.value[blockIdx].getHtmlContent().replaceAll(/\<br.*?\>/g, '').replace('<p>', '').replace('</p>', '') + '</p>').replace('</strong><strong>', '').replace('</em><em>', '')
    setTimeout(() => {
      blockElements.value[blockIdx-1].setCaretPos(prevBlockContentLength)
      props.page.blocks.splice(blockIdx, 1)
    })
  } else if ([BlockType.H1, BlockType.H2, BlockType.H3].includes(props.page.blocks[blockIdx-1].type)) {
    const prevBlockContentLength = (props.page.blocks[blockIdx-1] as any).details.value.length
    props.page.blocks[blockIdx-1].details.value += blockElements.value[blockIdx].getTextContent()
    setTimeout(() => {
      blockElements.value[blockIdx-1].setCaretPos(prevBlockContentLength)
      props.page.blocks.splice(blockIdx, 1)
    })
  } else {
    props.page.blocks.splice(blockIdx-1, 1)
    setTimeout(() => blockElements.value[blockIdx-1].moveToStart())
  }
}

function split (blockIdx: number) {
  const caretPos = blockElements.value[blockIdx].getCaretPos()
  insertBlock(blockIdx, '')
  props.page.blocks[blockIdx+1].details.value = (caretPos.tag ? `<p><${caretPos.tag}>` : '<p>') + props.page.blocks[blockIdx].details.value?.slice(caretPos.pos)
  if (isTextBlock(props.page.blocks[blockIdx].type)) {
    props.page.blocks[blockIdx].details.value = props.page.blocks[blockIdx].details.value?.slice(0, caretPos.pos) + (caretPos.tag ? `</${caretPos.tag}></p>` : '</p>')
  } else {
    props.page.blocks[blockIdx].details.value = props.page.blocks[blockIdx].details.value?.slice(0, caretPos.pos) + (caretPos.tag ? `</${caretPos.tag}></p>` : '')
  }
  setTimeout(() => blockElements.value[blockIdx+1].moveToStart())
}
</script>
