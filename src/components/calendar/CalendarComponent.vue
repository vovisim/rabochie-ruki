<script setup lang="ts">
import { computed, ref } from 'vue';
import { ArrowLeft, ArrowRight } from 'lucide-vue-next';

const { date, locale = 'ru' } = defineProps<{
  locale?: string; // язык
  date?: string; // дата
}>();

const emits = defineEmits<{
  (name: 'selectDate', e: Date): void; // устанавливает дату в родителе
}>();

const currentDate = ref<Date>(date ? new Date(date) : new Date());
const FirstDay = new Date(2025, 10, 3); // Понедельник
const selectedDate = ref<Date>(new Date(currentDate.value));
const currentMonth = ref<number>(currentDate.value.getMonth());
const currentYear = ref<number>(currentDate.value.getFullYear());

// вспомогательная функция для добавления дня
const addDays = (date: Date, days: number) => {
  return new Date(date.getFullYear(), date.getMonth(), date.getDate() + days);
};

// получаем названия дней недели в нужной нам локализации
const localeWeekdays = computed(() => {
  return [...Array(7)].map((_, i) =>
    new Intl.DateTimeFormat(locale, { weekday: 'short' }).format(addDays(FirstDay, i)),
  );
});

// Получаем дни недели включая остаток предыдущего месяца
const daysInMonth = computed(() => {
  const firstDay = new Date(currentYear.value, currentMonth.value, 1);
  const lastDay = new Date(currentYear.value, currentMonth.value + 1, 0);
  const days = [];

  // остаток предыдущего месяца
  for (let i = firstDay.getDay() - 1; i > 0; i--) {
    days.push(new Date(currentYear.value, currentMonth.value, 1 - i));
  }

  for (let i = 1; i <= lastDay.getDate(); i++) {
    days.push(new Date(currentYear.value, currentMonth.value, i));
  }

  return days;
});

// месяц назад
const previousMonth = (): void => {
  if (currentMonth.value === 0) {
    currentMonth.value = 11;
    currentYear.value--;
    return;
  }

  currentMonth.value--;
};

// месяц вперед
const nextMonth = (): void => {
  if (currentMonth.value === 11) {
    currentMonth.value = 0;
    currentYear.value++;
    return;
  }

  currentMonth.value++;
};

// устанавливаем дату и отправляем в родитель
const selectDate = (day: Date) => {
  selectedDate.value = day;
  emits('selectDate', day);
};

// получаем длинное название месяца
const monthName = computed(() =>
  new Intl.DateTimeFormat(locale, { month: 'long' }).format(
    new Date(currentYear.value, currentMonth.value),
  ),
);

// проверка на сегодня
const isToday = (day: Date): boolean => {
  const now = new Date();
  return (
    day.getDate() === now.getDate() &&
    day.getMonth() === now.getMonth() &&
    day.getFullYear() === now.getFullYear()
  );
};

// проверка на выбор
const isSelected = (day: Date): boolean => {
  return (
    day.getDate() === selectedDate.value.getDate() &&
    day.getMonth() === selectedDate.value.getMonth() &&
    day.getFullYear() === selectedDate.value.getFullYear()
  );
};
</script>

<template>
  <div class="calendar">
    <div class="calendar__header">
      <div class="button" @click="previousMonth">
        <ArrowLeft />
      </div>
      <div class="">{{ monthName }} {{ currentYear }}</div>
      <div class="button" @click="nextMonth">
        <ArrowRight />
      </div>
    </div>
    <div class="calendar__table">
      <div class="calendar__weekdays">
        <div v-for="day in localeWeekdays" :key="day" class="weekday">{{ day }}</div>
      </div>
      <div class="calendar__days">
        <div
          v-for="day in daysInMonth"
          :key="day.toISOString()"
          :class="{ today: isToday(day), selected: isSelected(day) }"
          class="day"
          @click="selectDate(day)"
        >
          {{ day.getDate() }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.calendar {
  width: max-content;

  &__header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 10px;
  }

  &__table {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  &__weekdays {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    text-align: center;
    gap: 10px;
    padding-bottom: 10px;
    border-bottom: #a4a4a4 1px solid;
  }

  &__days {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    text-align: center;
  }

  .day {
    height: 15px;
    line-height: 15px;
    cursor: pointer;
    padding: 10px;
    border: transparent solid 1px;
  }

  .day.today {
    color: #00ff9f;
  }

  .day.selected {
    border: #1ecaf1 solid 1px;
  }
}
</style>
