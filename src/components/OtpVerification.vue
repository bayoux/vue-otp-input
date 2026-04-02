<script setup lang="ts">
import { ref, onUnmounted } from 'vue'
import OtpInput from './OtpInput.vue'

const props = defineProps<{
  expiryTime?: number
}>()

const emit = defineEmits<{
  (e: 'submit', data: string): void
  (e: 'resend'): void
}>()

const secondsLeft = ref(props.expiryTime || 60)
const canResend = ref(false)
const otpValue = ref('')

let timer: number | null = null

const startTimer = () => {
  canResend.value = false
  secondsLeft.value = props.expiryTime || 60

  if (timer) clearInterval(timer)

  timer = setInterval(() => {
    if (secondsLeft.value > 0) {
      secondsLeft.value--
    } else {
      canResend.value = true
      if (timer) clearInterval(timer)
    }
  }, 1000)
}

const handleResend = () => {
  if (!canResend.value) return
  emit('resend')
  startTimer()
}

const onComplete = (code: string) => {
  emit('submit', code)
}

startTimer()

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<template>
  <div class="otp-verification">
    <OtpInput v-model="otpValue" :disabled="false" divider @complete="onComplete" />

    <div class="otp-verification__actions">
      <p v-if="!canResend" class="timer-text">
        Повторная отправка через <span>{{ secondsLeft }}с</span>
      </p>

      <button v-else class="resend-button" @click="handleResend">Отправить код повторно</button>
    </div>
  </div>
</template>

<style lang="scss">
.otp-verification {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;

  .timer-text {
    color: #64748b;
    font-size: 14px;
  }
  .resend-button {
    background: none;
    border: none;
    color: #3b82f6;
    font-weight: 600;
    cursor: pointer;
    text-decoration: underline;
  }
}
</style>
