<script setup lang="ts">
import { ref, watch, nextTick, onBeforeUpdate, onMounted } from 'vue'

interface Props {
  length?: number
  type?: 'text' | 'number'
  disabled?: boolean
  error?: boolean
  isPassword?: boolean
  autofocus?: boolean
  divider?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  length: 6,
  type: 'text',
  disabled: false,
  error: false,
  isPassword: false,
  autofocus: false,
  divider: false,
})

const modelValue = defineModel<string>({ default: '' })

const emit = defineEmits<{
  (e: 'complete', data: string): void
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
  () => props.error,
  (isError) => {
    if (isError) {
      inputRefs.value[0]?.focus()
      inputRefs.value[0]?.select()
    }
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

watch(modelValue, (newVal) => {
  if (!newVal) {
    digits.value = Array(props.length).fill('')
  }
})

onBeforeUpdate(() => {
  inputRefs.value = []
})

onMounted(() => {
  if (props.autofocus) {
    inputRefs.value[0]?.focus()
  }
})

const handleInput = (index: number, event: Event) => {
  const input = event.target as HTMLInputElement
  const val = input.value

  if (val.length > 1) {
    const chars = val.split('').slice(0, props.length - index)
    chars.forEach((char, i) => {
      if (index + i < props.length) digits.value[index + i] = char
    })
    const nextIdx = Math.min(index + chars.length, props.length - 1)
    inputRefs.value[nextIdx]?.focus()
    return
  }

  if (props.type === 'number' && !/^\d$/.test(val)) {
    digits.value[index] = ''
    return
  }

  digits.value[index] = val

  if (val && index < props.length - 1) {
    inputRefs.value[index + 1]?.focus()
  }
}

const handleKeyDown = (index: number, event: KeyboardEvent) => {
  if (props.type === 'number' && ['e', 'E', '+', '-', '.', ','].includes(event.key)) {
    event.preventDefault()
  }

  if (event.key === 'Backspace') {
    event.preventDefault()
    if (digits.value[index]) {
      digits.value[index] = ''
    } else if (index > 0) {
      digits.value[index - 1] = ''
      inputRefs.value[index - 1]?.focus()
    }
  } else if (event.key === 'ArrowLeft' && index > 0) {
    inputRefs.value[index - 1]?.focus()
  } else if (event.key === 'ArrowRight' && index < props.length - 1) {
    inputRefs.value[index + 1]?.focus()
  }
}

const handlePaste = (event: ClipboardEvent) => {
  if (props.disabled) return
  event.preventDefault()

  const pastedData = event.clipboardData?.getData('text') || ''
  const cleanData = props.type === 'number' ? pastedData.replace(/\D/g, '') : pastedData

  cleanData
    .split('')
    .slice(0, props.length)
    .forEach((char, i) => {
      digits.value[i] = char
    })

  const lastIndex = Math.min(cleanData.length, props.length - 1)
  nextTick(() => inputRefs.value[lastIndex]?.focus())
}

const dividerShow = (index: number) => {
  return index === Math.floor(props.length / 2) - 1 && props.length > 4
}
</script>

<template>
  <div
    class="otp-input"
    :class="{
      'otp-input--disabled': disabled,
      'otp-input--error': error,
    }"
  >
    <div class="otp-input__container" @paste="handlePaste">
      <template v-for="(_, index) in length" :key="index">
        <input
          ref="inputRefs"
          v-model="digits[index]"
          :type="isPassword ? 'password' : 'text'"
          :inputmode="type === 'number' ? 'numeric' : 'text'"
          :pattern="type === 'number' ? '[0-9]*' : undefined"
          :disabled="disabled"
          :aria-label="`Digit ${index + 1}`"
          autocomplete="one-time-code"
          class="otp-input__field"
          :class="{
            'otp-input__field--active': digits[index],
            'otp-input__field--error': error,
          }"
          maxlength="1"
          @input="handleInput(index, $event)"
          @keydown="handleKeyDown(index, $event)"
          @focus="($event.target as HTMLInputElement).select()"
        />

        <span v-if="divider && dividerShow(index)" class="otp-input__divider"> &mdash; </span>
      </template>
    </div>
  </div>
</template>

<style lang="scss">
.otp-input {
  display: flex;
  justify-content: center;

  &--disabled {
    opacity: 0.6;
    pointer-events: none;
  }

  &--error {
    animation: otp-shake 0.4s reverse;
  }

  &__container {
    display: flex;
    align-items: center;
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

    &::selection {
      background: rgba(59, 130, 246, 0.3);
    }

    &--active {
      border-color: #3b82f6;
    }

    &--error {
      border-color: #ef4444 !important;
      &:focus {
        box-shadow: 0 0 0 2px rgba(239, 68, 68, 0.2);
      }
    }

    &::-webkit-outer-spin-button,
    &::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }
  }

  &__divider {
    font-size: 20px;
    color: #cbd5e1;
    font-weight: normal;
  }
}

@keyframes otp-shake {
  0%,
  100% {
    transform: translateX(0);
  }
  20%,
  60% {
    transform: translateX(-6px);
  }
  40%,
  80% {
    transform: translateX(6px);
  }
}
</style>
