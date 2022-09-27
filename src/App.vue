<script setup lang="ts">
import datePicker from '@/components/date-picker.vue';
import dayjs from 'dayjs';
import { ref } from 'vue';
const aa = ref(new Date('1993-1-09'));
console.log(aa, dayjs());
</script>

<template>
  <div class="center">
    {{ aa }}
    <date-picker v-model:modelDate="aa" mode="range"> </date-picker>
  </div>
</template>

<style lang="css">
body {
  color: #1b1b1b;
  box-sizing: border-box;
}
.center {
  display: flex;
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
