<script setup lang="ts">
import { ref } from 'vue';
import CalendarComponent from '@/components/calendar/CalendarComponent.vue';

const currentDate = ref<Date>();
const currentLocale = ref<string>('ru');
const languages: Record<string, string> = {
  ru: 'Русский',
  en: 'English',
  de: 'Deutsch',
  fr: 'Français',
};
</script>

<template>
  <div>
    <select id="language" v-model="currentLocale">
      <option v-for="(label, code) in languages" :key="code" :value="code">
        {{ label }}
      </option>
    </select>
    <CalendarComponent
      date="2025-11-05"
      :locale="currentLocale"
      @select-date="currentDate = $event"
    />
  </div>
</template>

<style lang="scss">
body {
  background: #080808;
  color: #fff;
}

.button {
  cursor: pointer;

  &:hover {
    transition:
      color 0.25s ease,
      scale 0.25s ease;
    scale: 110%;
    color: #a4a4a4;
  }
}

select {
  margin-bottom: 40px;
  background: #000;
  color: #fff;
}
</style>
