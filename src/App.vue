<template>
  <div component type="theme" v-html="theme"></div>
  <Navigation @change-colors="onChangeColors" @change-theme="onChangeTheme" />
  <WeekSchedule size="16rem" :records="records" />
</template>

<script lang="ts" setup>
import WeekSchedule from './components/WeekSchedule.vue';
import Navigation from './components/Navigation.vue';
import { ref, computed } from 'vue';

const primary = ref('#000');

function onChangeColors(newcolors: any) {
  colors.value = newcolors;
}

function onChangeTheme(pallete: any) {
  theming.value = pallete;
}

const colors = ref(['#de7d14', '#de7d1472', '#bd357e', '#bd357e72']);
const theming = ref(['#fff', '#2c3e50']);

const theme = computed(() => {
  return `
    <style>
      :root {
        --primary: ${colors.value[0]};
        --primary-light: ${colors.value[1]};
        --secondary: ${colors.value[2]};
        --secondary-light: ${colors.value[3]};
        --background: ${theming.value[0]};
        --detail: ${theming.value[1]};
      }
    </style>`;
});

const records = [
  {
    description: 'lorem',
    date: new Date('May 30, 2022 12:00:00'),
    duration: 2,
  },
  {
    description: 'ipsum',
    date: new Date('Jun 02, 2022 13:00:00'),
    duration: 3,
  },
  {
    description: 'dolor',
    date: new Date('May 21, 2022 13:00:00'),
    duration: 5,
  },
  { description: 'sit', date: new Date('May 26, 2022 13:00:00'), duration: 1 },
  { description: 'amet', date: new Date('May 27, 2022 13:00:00'), duration: 4 },
  {
    description: 'consectetur',
    date: new Date('May 09, 2022 13:00:00'),
    duration: 5,
  },
  {
    description: 'adipisicing',
    date: new Date('May 28, 2022 13:00:00'),
    duration: 2,
  },
  { description: 'elit', date: new Date('Apr 28, 2022 11:00:00'), duration: 2 },
  {
    description: 'cumque',
    date: new Date('Apr 28, 2022 13:00:00'),
    duration: 6,
  },
  {
    description: 'numquam',
    date: new Date('Apr 25, 2022 14:00:00'),
    duration: 4,
  },
  {
    description: 'impedit',
    date: new Date('May 10, 2022 12:00:00'),
    duration: 3,
  },
];

const workHours = ['11:00', '12:00', '13:00', '14:00', '15:00'];
</script>

<style>
body {
  margin: 0;
  background-color: var(--background);
  color: var(--detail);
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
