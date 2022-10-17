<script setup lang="ts">
import dayjs from 'dayjs';
import ObjeceSupport from 'dayjs/plugin/objectSupport';
import IsBetween from 'dayjs/plugin/isBetween';
import { useCounter } from '@vueuse/core';
import { ref, computed, popScopeId } from 'vue';
import type { DatePickerMode } from './types';

dayjs.extend(IsBetween);
dayjs.extend(ObjeceSupport);

type DateType = 'normal' | 'active' | 'week' | 'next-month' | 'prev-month';
interface DateTableData {
  year: number;
  month: number;
  date: number;
  disabled: boolean;
  type: DateType;
}

const props = defineProps<{
  mode: DatePickerMode;
  internalModelDate: string | string[];
  currentViewDate: string;
}>();

const emits = defineEmits<{ (e: 'setInternalValue', date: string): void }>();

const firstDayOfWeek = 0;

const weekList = computed(() => {
  const weekList = ['S', 'M', 'T', 'W', 'T', 'F', 'S'];
  // const weekList = ['日', '一', '二', '三', '四', '五', '六'];
  for (let i = 0; i < firstDayOfWeek; i++) {
    const week = weekList.shift();
    weekList.push(week!);
  }
  return weekList;
});

// const dateList = ref<DateTableData[]>([]);

const dateList = computed<DateTableData[]>(() => {
  const year = +dayjs(props.currentViewDate).format('YYYY');
  const month = +dayjs(props.currentViewDate).format('MM');
  const monthFirstDayOfWeek =
    dayjs(props.currentViewDate).startOf('month').day() - firstDayOfWeek;
  const endDateOfMonth = dayjs(props.currentViewDate).endOf('month').date();
  const nextMonth = dayjs(props.currentViewDate).add(1, 'month');
  const prevMonth = dayjs(props.currentViewDate).subtract(1, 'month');
  const endDateOfPrevMonth = prevMonth.endOf('month').date();
  const { count: prevMonthDate, inc: incPrevMonthDate } = useCounter(
    endDateOfPrevMonth + 1 - monthFirstDayOfWeek
  );
  const { count: nowMonthDate, inc: incNowMonthDate } = useCounter(1);
  const { count: nextMonthDate, inc: incNextMonthDate } = useCounter(1);
  const dateList: DateTableData[] = [];
  for (let i = 0; i < 42; i++) {
    if (monthFirstDayOfWeek <= i && nowMonthDate.value <= endDateOfMonth) {
      dateList.push({
        year: year,
        month: month,
        date: nowMonthDate.value,
        disabled: disabledDate(
          new Date(
            dayjs(`${year}-${month}-${nowMonthDate.value}`).format('YYYY-MM-DD')
          )
        )
          ? true
          : false,
        type: handlerPickDateType(
          dayjs(`${year}-${month}-${nowMonthDate.value}`).format('YYYY-MM-DD')
        ),
      });
      incNowMonthDate();
      continue;
    }
    if (monthFirstDayOfWeek <= i && monthFirstDayOfWeek + endDateOfMonth <= i) {
      dateList.push({
        year: nextMonth.year(),
        month: nextMonth.month(),
        date: nextMonthDate.value,
        disabled: true,
        type: 'next-month' as DateType,
      });
      incNextMonthDate();
      continue;
    }
    dateList.push({
      year: prevMonth.year(),
      month: prevMonth.month(),
      date: prevMonthDate.value,
      disabled: true,
      type: 'prev-month' as DateType,
    });
    incPrevMonthDate();
  }
  return dateList;
});

function handlerPickDateType(dateString: string): DateType {
  let type: DateType = 'normal';
  if (Array.isArray(props.internalModelDate)) {
    if (props.internalModelDate.includes(dateString)) {
      type = 'active';
    }
    if (props.mode === 'range' && props.internalModelDate.length === 2) {
      if (
        dayjs(dateString).isBetween(
          props.internalModelDate[0],
          props.internalModelDate[1],
          null,
          '()'
        )
      ) {
        type = 'active';
      }
    }
  } else {
    if (props.internalModelDate === dateString) {
      type = 'active';
    }
  }
  return type;
}

function clickDateBtn(date: DateTableData) {
  emits(
    'setInternalValue',
    dayjs(`${date.year}-${date.month}-${date.date}`).format('YYYY-MM-DD')
  );
}

const disabledDate = (date: Date) => {
  // if (date.getDate() % 2) {
  //   return true;
  // }
  return false;
};
</script>

<template>
  <div class="date-table__body">
    <div class="date-table__week">
      <span v-for="week in weekList" :key="week">{{ week }}</span>
    </div>
    <div class="date-table__date">
      <button
        v-for="(date, index) in dateList"
        :key="index"
        :class="{
          'date-btn__disabled': date.disabled,
          'is-active': date.type === 'active',
          'is-left-active': dateList[index - 1]?.type === 'active',
          'is-right-active': dateList[index + 1]?.type === 'active',
        }"
        :disabled="date.disabled && date.type !== 'normal'"
        @click="clickDateBtn(date)"
      >
        {{ date.date }}
      </button>
    </div>
  </div>
</template>

<style lang="css" scoped>
.date-table__body {
  padding: 0px 12px;
}

.date-table__week {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  justify-items: center;
  height: 32px;
  align-items: center;
}
.date-table__week > span {
  opacity: 70%;
}

.date-table__date {
  display: grid;
  justify-content: center;
  grid-template-columns: repeat(7, 1fr);
  grid-template-rows: repeat(6, 32px);
  justify-items: center;
  align-items: center;
  row-gap: 4px;
}

.date-table__date > button {
  width: 100%;
  height: 100%;
  border-radius: 100%;
}

.date-table__date > button:hover {
  background-color: rgba(0, 0, 0, 0.2);
}
.date-table__date > button.is-active:hover,
.is-active {
  background-color: #ff9000;
}

.is-left-active {
  border-bottom-left-radius: 0% !important;
  border-top-left-radius: 0% !important;
}

.is-right-active {
  border-bottom-right-radius: 0% !important;
  border-top-right-radius: 0% !important;
}

.date-btn__disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.date-table__date > .date-btn__disabled:hover {
  background-color: transparent;
}
</style>
