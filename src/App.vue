<script setup lang="ts">
import datePicker from '@/components/date-picker.vue';
import dayjs from 'dayjs';
import { ref } from 'vue';
import type { DatePickerMode } from './components/types';
const aa = ref(new Date('1993-1-09'));
const mode = ref<DatePickerMode>('only');
</script>

<template>
  <div class="center">
    <div>Selected: {{ mode }}</div>

    <select v-model="mode">
      <option disabled value="">Please select one</option>
      <option>A</option>
      <option>B</option>
      <option>C</option>
    </select>
    <date-picker v-model:modelDate="aa" mode="multiple"> </date-picker>
  </div>
</template>

<style lang="css">
body {
  color: #1b1b1b;
  box-sizing: border-box;
}
.center {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
button {
  appearance: none;
  border: none;
  padding: 0px;
  background-color: transparent;
  cursor: pointer;
}
</style>
