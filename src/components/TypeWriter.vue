<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

interface Props {
  texts: string[]
  typeSpeed?: number
  deleteSpeed?: number
  pauseDuration?: number
  className?: string
}

const props = withDefaults(defineProps<Props>(), {
  typeSpeed: 100,
  deleteSpeed: 50,
  pauseDuration: 2000,
  className: ''
})

const displayText = ref('')
const currentIndex = ref(0)
const isDeleting = ref(false)
const isPaused = ref(false)
let timeoutId: NodeJS.Timeout | null = null

const typeWriter = () => {
  const currentText = props.texts[currentIndex.value]
  
  if (isPaused.value) {
    timeoutId = setTimeout(() => {
      isPaused.value = false
      isDeleting.value = true
      typeWriter()
    }, props.pauseDuration)
    return
  }

  if (isDeleting.value) {
    displayText.value = currentText.substring(0, displayText.value.length - 1)
    
    if (displayText.value === '') {
      isDeleting.value = false
      currentIndex.value = (currentIndex.value + 1) % props.texts.length
      timeoutId = setTimeout(typeWriter, props.typeSpeed)
    } else {
      timeoutId = setTimeout(typeWriter, props.deleteSpeed)
    }
  } else {
    displayText.value = currentText.substring(0, displayText.value.length + 1)
    
    if (displayText.value === currentText) {
      isPaused.value = true
      typeWriter()
    } else {
      timeoutId = setTimeout(typeWriter, props.typeSpeed)
    }
  }
}

onMounted(() => {
  typeWriter()
})

onUnmounted(() => {
  if (timeoutId) {
    clearTimeout(timeoutId)
  }
})
</script>

<template>
  <span :class="className">
    {{ displayText }}
    <span class="animate-pulse">|</span>
  </span>
</template>