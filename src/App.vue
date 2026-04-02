<script setup lang="ts">
import { ref } from 'vue'
import OtpVerification from './components/OtpVerification.vue'

const isLoading = ref(false)
const hasError = ref(false)

const verifyCode = async (code: string) => {
  isLoading.value = true
  hasError.value = false

  try {
    console.log('Sending code:', code)
    await new Promise((resolve) => setTimeout(resolve, 2000))

    if (code !== '123456') throw new Error('Invalid code')

    alert('Успешно!')
  } catch {
    hasError.value = true
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="container">
    <div class="auth-container">
      <h2>Введите код из SMS</h2>

      <OtpVerification
        :error="hasError"
        :expiry-time="10"
        @submit="verifyCode"
        @resend="
          () => {
            hasError = false
            console.log('Resending...')
          }
        "
      />

      <div v-if="isLoading" class="loader">Проверка...</div>
      <p v-if="hasError" class="error-msg">Неверный код. Попробуйте еще раз.</p>
    </div>
  </div>
</template>

<style lang="scss">
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}

.auth-container {
  padding: 40px;
  text-align: center;
}
.loader {
  margin-top: 15px;
  color: #3b82f6;
}
.error-msg {
  margin-top: 15px;
  color: #ef4444;
  font-size: 14px;
}
</style>
