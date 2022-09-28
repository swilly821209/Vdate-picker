<script setup lang="ts">
import { onMounted, ref, defineProps, computed } from 'vue';
import dayjs from 'dayjs';

const props = defineProps<{
  currentViewDate: string;
  min: string;
  max: string;
}>();

const emits = defineEmits<{ (e: 'setYear', year: number): void }>();

const yearListItem = ref<HTMLDivElement | null>(null);

const yearList = computed<number[]>(() => {
  const yearList = [];
  const nowYear = +dayjs(props.currentViewDate).format('YYYY');
  const maxYear = props.max
    ? +dayjs(props.max).format('YYYY')
    : Number(nowYear) + 100;
  const minYear = props.min
    ? +dayjs(props.min).format('YYYY')
    : Number(nowYear) - 100;

  for (let year = maxYear; year >= minYear; year--) {
    yearList.push(year);
  }
  return yearList;
});

onMounted(() => {
  if (yearListItem.value) {
    yearListItem.value.scrollTop =
      yearListItem.value.scrollHeight / 2 - 32 * 4 - 16;
  }
});
</script>

<template>
  <div class="year-table__body" ref="yearListItem">
    <button
      v-for="year in yearList"
      :key="year"
      :class="{
        'is-active': +dayjs(props.currentViewDate).format('YYYY') === year,
      }"
      @click="emits('setYear', year)"
    >
      {{ year }}
    </button>
  </div>
</template>

<style lang="css" scoped>
.year-table__body {
  padding: 0 12px;
  height: 284px;
  overflow-y: auto;
}
.year-table__body button {
  display: block;
  width: 100%;
  height: 32px;
  text-align: center;
}

.is-active {
  color: #ff9000;
  font-size: 16px;
}
</style>
