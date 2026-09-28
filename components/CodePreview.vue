<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as monaco from 'monaco-editor'

const props = defineProps({
  modelValue: { type: String, default: '' },
  lang: { type: String, default: 'html' },
  height: { type: String, default: '420px' },
  editorLabel: { type: String, default: 'Edit this code ↓' },
  previewLabel: { type: String, default: 'Live result ↓' },
  showCode: { type: Boolean, default: true },
})

const emit = defineEmits(['update:modelValue', 'update:showCode'])

const editorContainer = ref(null)
let editorInstance = null

onMounted(() => {
  if (!props.showCode) return

  editorInstance = monaco.editor.create(editorContainer.value, {
    value: props.modelValue,
    language: props.lang,
    automaticLayout: true,
    minimap: { enabled: false },
    fontSize: 12,
    scrollBeyondLastLine: false,
  })

  editorInstance.onDidChangeModelContent(() => {
    emit('update:modelValue', editorInstance.getValue())
  })
})

// keep editor in sync if the parent changes modelValue externally
watch(() => props.modelValue, (val) => {
  if (editorInstance && val !== editorInstance.getValue()) {
    editorInstance.setValue(val)
  }
})

onUnmounted(() => editorInstance?.dispose())
</script>

<template>
    <button @click="emit('update:showCode', !showCode)" class="border border-gray-400/50 rounded-lg mb-1 pl-1 pr-1">{{showCode? 'hide' : 'show'}}  code</button>
  <div class="grid gap-4" :class="showCode ? 'grid-cols-2' : 'grid-cols-1'">
    <div v-show="showCode"  class="border border-gray-400/50 rounded-lg p-3">
      <span class="text-xs opacity-50">{{ editorLabel }}</span>
      <div 
        ref="editorContainer" 
        :style="{ height, borderRadius: '6px', overflow: 'auto' }"
        @keydown.stop
        @keyup.stop
        @keypress.stop 
      />
    </div>

    <div class="border border-gray-400/50 rounded-lg p-3">
      <span class="text-xs opacity-50">{{ previewLabel }}</span>
      <div v-html="modelValue" class="flex gap-3 items-start flex-wrap mt-2" />
    </div>
  </div>
</template>