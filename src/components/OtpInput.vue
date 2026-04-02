<script setup lang="ts">
import { ref, watch, nextTick } from 'vue'

interface Props {
  length?: number
  type?: 'text' | 'number'
}

const props = withDefaults(defineProps<Props>(), {
  length: 6,
  type: 'text',
})

const modelValue = defineModel<string>({ default: '' })
const emit = defineEmits<{
  (e: 'complete', value: string): void
}>()

const digits = ref<string[]>(Array.from({ length: props.length }, () => ''))
const inputRefs = ref<HTMLInputElement[]>([])

watch(
  () => props.length,
  (newLen) => {
    digits.value = Array.from({ length: newLen }, () => '')
  },
)

watch(
  digits,
  (newVal) => {
    const code = newVal.join('')
    modelValue.value = code

    if (code.length === props.length && newVal.every((d) => d !== '')) {
      emit('complete', code)
    }
  },
  { deep: true },
)

const handleInput = (index: number, event: Event) => {
  const input = event.target as HTMLInputElement
  let val = input.value.slice(-1)

  if (props.type === 'number' && !/^\d$/.test(val)) {
    val = ''
  }

  digits.value[index] = val

  if (val && index < props.length - 1) {
    inputRefs.value[index + 1]?.focus()
  }
}

const handleKeyDown = (index: number, event: KeyboardEvent) => {
  switch (event.key) {
    case 'Backspace':
      if (!digits.value[index] && index > 0) {
        inputRefs.value[index - 1]?.focus()
      }
      break
    case 'ArrowLeft':
      if (index > 0) inputRefs.value[index - 1]?.focus()
      break
    case 'ArrowRight':
      if (index < props.length - 1) inputRefs.value[index + 1]?.focus()
      break
  }
}

const handlePaste = (event: ClipboardEvent) => {
  event.preventDefault()
  const pastedData = event.clipboardData?.getData('text') || ''
  const cleanData = props.type === 'number' ? pastedData.replace(/\D/g, '') : pastedData

  const data = cleanData.slice(0, props.length).split('')

  data.forEach((char, i) => {
    if (i < props.length) digits.value[i] = char
  })

  const nextIndex = Math.min(data.length, props.length - 1)
  nextTick(() => {
    inputRefs.value[nextIndex]?.focus()
  })
}
</script>

<template>
  <div class="otp-input">
    <div class="otp-input__container" @paste="handlePaste">
      <input
        v-for="(_, index) in length"
        :key="index"
        ref="inputRefs"
        v-model="digits[index]"
        type="text"
        :inputmode="type === 'number' ? 'numeric' : 'text'"
        autocomplete="one-time-code"
        class="otp-input__field"
        :class="{
          'otp-input__field--active': digits[index],
          'otp-input__field--filled': digits[index],
        }"
        maxlength="1"
        @input="handleInput(index, $event)"
        @keydown="handleKeyDown(index, $event)"
        @focus="($event.target as HTMLInputElement).select()"
      />
    </div>
  </div>
</template>

<style lang="scss">
.otp-input {
  $root: &;

  display: flex;
  justify-content: center;

  &__container {
    display: flex;
    gap: 12px;
  }

  &__field {
    width: 48px;
    height: 56px;
    text-align: center;
    font-size: 24px;
    font-weight: bold;
    border: 2px solid #e2e8f0;
    border-radius: 8px;
    background-color: #ffffff;
    transition: all 0.2s ease-in-out;
    outline: none;

    &:focus {
      border-color: #3b82f6;
      box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2);
    }

    &--active {
      border-color: #3b82f6;
    }

    &::-webkit-outer-spin-button,
    &::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }
  }
}
</style>
