<script setup lang="ts">
import datePicker from '@/components/date-picker.vue';
import dayjs from 'dayjs';
import { ref } from 'vue';
import type { DatePickerMode } from './components/types';
const a = ref(new Date());
const b = ref([new Date(), new Date('2022-10-10')]);
const c = ref([new Date()]);
</script>

<template>
  <div class="center">
    <div class="date-show">
      <span>mode: <strong>only</strong></span>
      <span>modelValue:</span>
      <span>{{ a }}</span>
      <date-picker v-model:modelDate="a" mode="only"> </date-picker>
    </div>
    <div class="date-show">
      <span>mode: <strong>multiple</strong></span>
      <span>modelValue:</span>
      <span>{{ b }}</span>
      <date-picker v-model:modelDate="b" mode="multiple"> </date-picker>
    </div>
    <div class="date-show">
      <span>mode: <strong>range</strong></span>
      <span>modelValue:</span>
      <span>{{ c }}</span>
      <date-picker v-model:modelDate="c" mode="range"> </date-picker>
    </div>
  </div>
</template>

<style lang="css" scoped>
body {
  color: #1b1b1b;
  box-sizing: border-box;
}
.center {
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  height: 100vh;
}
.date-show {
  width: 248px;
  display: flex;
  flex-direction: column;
}
@media (max-width: 639px) {
  .center {
    height: auto;
    flex-direction: column;
  }
  .date-show {
    margin-bottom: 10px;
  }
}
</style>
