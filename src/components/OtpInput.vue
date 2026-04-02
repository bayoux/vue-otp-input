<script setup lang="ts">
import { ref, watch } from 'vue'

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
  digits,
  (newVal) => {
    const code = newVal.join('')
    modelValue.value = code

    if (code.length === props.length && newVal.every((d) => d !== '')) {
      console.log('Emit complete event with:', code)
      emit('complete', code)
    }
  },
  { deep: true },
)

const handleInput = (index: number, event: Event) => {
  const input = event.target as HTMLInputElement
  const val = input.value.slice(-1)

  digits.value[index] = val

  if (val && index < props.length - 1) {
    inputRefs.value[index + 1]?.focus()
  }
}

const handleKeyDown = (index: number, event: KeyboardEvent) => {
  if (event.key === 'Backspace' && !digits.value[index] && index > 0) {
    inputRefs.value[index - 1]?.focus()
  }
}

const handlePaste = (event: ClipboardEvent) => {
  event.preventDefault()
  const pastedData = event.clipboardData?.getData('text') || ''
  const data = pastedData.slice(0, props.length).split('')

  data.forEach((char, i) => {
    if (i < props.length) digits.value[i] = char
  })

  const nextIndex = Math.min(data.length, props.length - 1)
  inputRefs.value[nextIndex]?.focus()
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
        inputmode="numeric"
        autocomplete="one-time-code"
        class="otp-input__field"
        :class="{ 'otp-input__field--active': digits[index] }"
        maxlength="1"
        @input="handleInput(index, $event)"
        @keydown="handleKeyDown(index, $event)"
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
