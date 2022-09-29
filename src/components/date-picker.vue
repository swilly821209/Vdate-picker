<script setup lang="ts">
import { computed, shallowRef, withDefaults, ref, watch } from 'vue';
import IconArrowRight from '@/components/icon/IconArrowRight.vue';
import IconArrowLeft from '@/components/icon/IconArrowLeft.vue';
import DateTable from '@/components/date-table.vue';
import MonthTable from '@/components/month-table.vue';
import YearTable from '@/components/year-table.vue';
import dayjs from 'dayjs';
import { useVModel } from '@vueuse/core';
import type { DatePickerMode } from './types';

type TableName = 'Date' | 'Month' | 'Year';

const props = withDefaults(
  defineProps<{
    modelDate?: Date | Date[];
    max?: string;
    min?: string;
    mode?: DatePickerMode;
  }>(),
  {
    modelDate: () => new Date(),
    max: '',
    min: '',
    mode: 'only',
  }
);
const emits = defineEmits<{ (e: 'update:modelDate', value: Date): void }>();

const modelDateValue = useVModel(props, 'modelDate', emits);

const internalModelDate = ref<string | string[]>(
  Array.isArray(modelDateValue.value) ? [] : ''
);

if (Array.isArray(modelDateValue.value)) {
  internalModelDate.value = modelDateValue.value.map(date => {
    return dayjs(date).format('YYYY-MM-DD');
  });
} else {
  internalModelDate.value = dayjs(modelDateValue.value).format('YYYY-MM-DD');
}

watch(internalModelDate, internalModelDate => {
  if (Array.isArray(internalModelDate)) {
    internalModelDate.forEach(value => {
      if (Array.isArray(modelDateValue.value)) {
        modelDateValue.value.push(new Date(value));
      }
    });
  } else {
    modelDateValue.value = new Date(internalModelDate);
  }
});

const activeDate = computed<string>(() => {
  return Array.isArray(internalModelDate.value)
    ? internalModelDate.value[internalModelDate.value.length - 1]
    : internalModelDate.value;
});

const currentViewDate = ref<string>(
  Array.isArray(internalModelDate.value)
    ? internalModelDate.value[internalModelDate.value.length - 1]
    : internalModelDate.value
);

const currentTable = ref<TableName>('Date');

const panelValue = computed(() => {
  switch (currentTable.value) {
    case 'Date':
      return dayjs(currentViewDate.value).format('MMMM YYYY');
    case 'Month':
      return dayjs(currentViewDate.value).format('YYYY');
    default:
      return '';
  }
});

function clickPanel() {
  if (currentTable.value === 'Date') {
    changeTable('Month');
  } else if (currentTable.value === 'Month') {
    changeTable('Year');
  } else {
    changeTable('Month');
  }
}

function changeTable(tableName: TableName) {
  currentTable.value = tableName;
}

function setCurrentViewYear(year: number) {
  currentViewDate.value = dayjs(currentViewDate.value)
    .set('y', year)
    .format('YYYY-MM-DD');

  changeTable('Month');
}

function setCurrentViewMonth(month: number) {
  currentViewDate.value = dayjs(currentViewDate.value)
    .set('M', month)
    .format('YYYY-MM-DD');

  changeTable('Date');
}

function moveYear(forward: boolean) {
  currentViewDate.value = forward
    ? dayjs(currentViewDate.value).add(1, 'year').format('YYYY-MM-DD')
    : dayjs(currentViewDate.value).subtract(1, 'year').format('YYYY-MM-DD');
}

function moveMonth(forward: boolean) {
  currentViewDate.value = forward
    ? dayjs(currentViewDate.value).add(1, 'month').format('YYYY-MM-DD')
    : dayjs(currentViewDate.value).subtract(1, 'month').format('YYYY-MM-DD');
}

function setModelDateValue(dateValue: Date) {
  if (props.mode === 'readonly') return;
  console.log(findIndexModelDate(dateValue));
  if (findIndexModelDate(dateValue) !== -1) {
    Array.isArray(modelDateValue.value) &&
      modelDateValue.value.splice(findIndexModelDate(dateValue), 1);
    return;
  }
  if (props.mode === 'range' || props.mode === 'multiple') {
    if (Array.isArray(modelDateValue.value)) {
      if (modelDateValue.value.length > 1 && props.mode === 'range') {
        modelDateValue.value.shift();
      }
      modelDateValue.value.push(dateValue);
      return;
    }
    modelDateValue.value = [dateValue];
    return;
  }
  modelDateValue.value = dateValue;
}

function findIndexModelDate(date: Date): number {
  console.log(modelDateValue.value);
  if (Array.isArray(modelDateValue.value)) {
    return modelDateValue.value.findIndex(modelDate => {
      console.log(dayjs(modelDate).format('YYYY-MM-DD'), 'f');
      return (
        dayjs(modelDate).format('YYYY-MM-DD') ===
        dayjs(date).format('YYYY-MM-DD')
      );
    });
  }
  return -1;
}
</script>
<template>
  <div class="date-picker">
    <div class="date-picker__title">
      <div class="date-picker-title__year">
        {{ dayjs(activeDate).format('YYYY') }}
      </div>
      <div class="date-picker-title__date">
        {{ dayjs(activeDate).format('ddd, MMM D') }}
      </div>
    </div>
    <div class="date-picker__panel" v-if="currentTable !== 'Year'">
      <button
        v-show="currentTable === 'Month'"
        class="date-picker-arrow__btn"
        @click="moveYear(false)"
      >
        <slot name="leftArrow">
          <IconArrowLeft />
        </slot>
      </button>
      <button
        v-show="currentTable === 'Date'"
        class="date-picker-arrow__btn"
        @click="moveMonth(false)"
      >
        <slot name="leftArrow">
          <IconArrowLeft />
        </slot>
      </button>
      <button class="date-picker-panel__value" @click="clickPanel">
        {{ panelValue }}
      </button>
      <button
        v-show="currentTable === 'Month'"
        class="date-picker-arrow__btn"
        @click="moveYear(true)"
      >
        <IconArrowRight />
      </button>
      <button
        v-show="currentTable === 'Date'"
        class="date-picker-arrow__btn"
        @click="moveMonth(true)"
      >
        <IconArrowRight />
      </button>
    </div>
    <YearTable
      :currentViewDate="currentViewDate"
      :max="max"
      :min="min"
      @setYear="setCurrentViewYear"
      v-if="currentTable === 'Year'"
      class="date-picker__body"
    />
    <MonthTable
      :currentViewDate="currentViewDate"
      :activeDate="activeDate"
      v-else-if="currentTable === 'Month'"
      class="date-picker__body"
      @setMonth="setCurrentViewMonth"
    />
    <DateTable
      :internalModelDate="internalModelDate"
      :currentViewDate="currentViewDate"
      :mode="mode"
      @setModelDateValue="setModelDateValue"
      v-else
      class="date-picker__body"
    />
  </div>
</template>

<style lang="css" scoped>
.date-picker__title {
  padding: 8px 12px;
  background-color: #ff9000;
}
.date-picker-title__year {
  margin-bottom: 4px;
}
.date-picker-title__date {
  font-size: 32px;
}
.date-picker__body {
  width: 224px;
}
.date-picker-arrow__btn {
  display: flex;
  justify-content: center;
  align-items: center;
}
.date-picker__panel {
  display: grid;
  grid-template-columns: 24px 1fr 24px;
  justify-items: center;
  width: 224px;
  padding: 8px 12px;
}
.date-picker-panel__value {
  font-size: 20px;
}
</style>
