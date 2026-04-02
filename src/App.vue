<script setup lang="ts">
import { ref, watch } from 'vue'
import OtpInput from './components/OtpInput.vue'

const otpCode = ref('')

watch(otpCode, (newVal) => {
  console.log('OTP Code changed:', newVal)
})

const handleOtpComplete = (code: string) => {
  console.log('Код введен полностью:', code)
}
</script>

<template>
  <div class="container">
    <OtpInput v-model="otpCode" :length="6" type="number" @complete="handleOtpComplete" />
  </div>
</template>

<style lang="scss">
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}
</style>
