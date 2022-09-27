<script setup lang="ts">
import dayjs from 'dayjs';
import ObjeceSupport from 'dayjs/plugin/objectSupport';
import IsBetween from 'dayjs/plugin/isBetween';
import { useCounter } from '@vueuse/core';
import { ref, computed } from 'vue';

dayjs.extend(IsBetween);
dayjs.extend(ObjeceSupport);
type DatePickerMode = 'only' | 'multiple' | 'range' | 'readonly';
type DateType = 'normal' | 'today' | 'week' | 'next-month' | 'prev-month';
interface DateTableData {
  year: number;
  month: number;
  date: number;
  disabled: boolean;
  type: DateType;
}
const props = defineProps<{
  date: Date;
  nowPickDate: Date;
  mode: DatePickerMode;
}>();
const emits = defineEmits<{ (e: 'setModelDateValue', date: Date): void }>();

const firstDayOfWeek = 0;

const weekList = computed(() => {
  // const weekList = ['S', 'M', 'T', 'W', 'T', 'F', 'S'];
  const weekList = ['日', '一', '二', '三', '四', '五', '六'];
  for (let i = 0; i < firstDayOfWeek; i++) {
    const week = weekList.shift();
    weekList.push(week!);
  }
  return weekList;
});

// const dateList = ref<DateTableData[]>([]);

const dateList = computed<DateTableData[]>(() => {
  const year = +dayjs(props.date).format('YYYY');
  const month = +dayjs(props.date).format('MM');
  const monthFirstDayOfWeek =
    dayjs(props.date).startOf('month').day() - firstDayOfWeek;
  const endDateOfMonth = dayjs(props.date).endOf('month').date();
  const nextMonth = dayjs(props.date).add(1, 'month');
  const prevMonth = dayjs(props.date).subtract(1, 'month');
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
          new Date(`${year}-${month}-${nowMonthDate.value}`)
        )
          ? true
          : false,
        type: handlerPickDayType(
          new Date(`${year}-${month}-${nowMonthDate.value}`)
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

function handlerPickDayType(date: Date): DatePickerMode {
  let type = 'normal';
  if (props.mode === 'range' && props.nowPickDate.length === 2) {
    dayjs(date).isBetween(props.nowPickDate[0], props.nowPickDate[1], 'day', '[]')
      ? (type = 'today')
      : (type = 'normal');
  }
  if (
    Array.isArray(props.nowPickDate) &&
    dayjs(props.nowPickDate).isSame(dayjs(date), 'day')
  ) {
    type = 'today';
  }
  return type as DatePickerMode;
}

const setDateTableData = (
  year: number,
  month: number,
  disabledDate: (date: Date) => boolean
): DateTableData[] => {
  const nowMonth = dayjs({ year: year, month: (month -= 1) });

  const monthFirstDayOfWeek = nowMonth.startOf('month').day() - firstDayOfWeek;
  const endDateOfMonth = nowMonth.endOf('month').date();
  const nextMonth = nowMonth.add(1, 'month');
  const prevMonth = nowMonth.subtract(1, 'month');
  const endDateOfPrevMonth = prevMonth.endOf('month').date();
  const { count: prevMonthDate, inc: incPrevMonthDate } = useCounter(
    endDateOfPrevMonth + 1 - monthFirstDayOfWeek
  );
  const { count: nowMonthDate, inc: incNowMonthDate } = useCounter(1);
  const { count: nextMonthDate, inc: incNextMonthDate } = useCounter(1);
  const dateList = [];
  for (let i = 0; i < 42; i++) {
    if (monthFirstDayOfWeek <= i && nowMonthDate.value <= endDateOfMonth) {
      dateList.push({
        year: year,
        month: month,
        date: nowMonthDate.value,
        disabled: disabledDate(new Date(year, month, nowMonthDate.value))
          ? true
          : false,
        type: 'normal' as DateType,
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
};
function clickDateBtn(date: DateTableData) {
  console.log(
    new Date(`${date.year}-${date.month}-${date.date}`),
    `${date.year}-${date.month}-${date.date}`
  );
  emits(
    'setModelDateValue',
    new Date(`${date.year}-${date.month}-${date.date}`)
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
          'is-active': date.type === 'today',
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

.date-table__date > button:hover,
.is-active {
  background-color: #ff9000;
}

.date-btn__disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.date-table__date > .date-btn__disabled:hover {
  background-color: transparent;
}
</style>
