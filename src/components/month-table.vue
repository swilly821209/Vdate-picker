<script setup lang="ts">
import { useVModel } from '@vueuse/core';
import dayjs from 'dayjs';
import { computed } from 'vue';

const props = defineProps<{ date: Date; nowPickDate: Date }>();
const emits = defineEmits<{ (e: 'setMonth', month: number): void }>();

const monthList = [
  'JAN',
  'FEB',
  'MAR',
  'APR',
  'MAY',
  'JUN',
  'JUL',
  'AUG',
  'SEP',
  'OCT',
  'NOV',
  'DEC',
];

const isSameYear = computed<boolean>(() => {
  return (
    dayjs(props.date).format('YYYY') === dayjs(props.nowPickDate).format('YYYY')
  );
});
</script>

<template>
  <div class="month-table__body">
    <button
      v-for="(month, index) in monthList"
      :key="month"
      :class="{
        'is-active':
          isSameYear &&
          dayjs(props.nowPickDate).format('MMM').toLocaleUpperCase() === month,
      }"
      @click="emits('setMonth', index)"
    >
      {{ month }}
    </button>
  </div>
</template>

<style lang="css" scoped>
.month-table__body {
  padding: 2px 12px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(4, 48px);
  row-gap: 16px;
  justify-items: center;
  align-items: center;
}
.month-table__body > button:hover,
.is-active {
  background-color: #ff9000;
}
.month-table__body > button {
  width: 100%;
  height: 100%;
  border-radius: 4px;
}
</style>
