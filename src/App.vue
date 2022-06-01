<template>
  <div component type="theme" v-html="theme"></div>
  <TopBar @change-colors="onChangeColors" @change-theme="onChangeTheme" />
  <WeekSchedule :minimal="windowWidth <= 700" size="16rem" :records="records" />
</template>

<script lang="ts" setup>
import WeekSchedule from './components/WeekSchedule.vue';
import TopBar from './components/TopBar.vue';
import { ref, computed, onMounted, onUnmounted } from 'vue';

const primary = ref('#000');
const windowWidth = ref(window.innerWidth);
const onWidthChange = () => (windowWidth.value = window.innerWidth);

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

const today = new Date();

const records = [
  {
    description: 'lorem',
    date: new Date(today.getFullYear(), today.getMonth(), today.getDate(), 13),
    duration: 2,
  },
  {
    description: 'ipsum',
    date: new Date(
      today.getFullYear(),
      today.getMonth(),
      today.getDate() + 1,
      13
    ),
    duration: 3,
  },
  {
    description: 'dolor',
    date: new Date(
      today.getFullYear(),
      today.getMonth(),
      today.getDate() + 2,
      13
    ),
    duration: 5,
  },
  {
    description: 'sit',
    date: new Date(
      today.getFullYear(),
      today.getMonth(),
      today.getDate() - 1,
      13
    ),
    duration: 5,
  },
  {
    description: 'amet',
    date: new Date(today.getFullYear(), today.getMonth(), today.getDate(), 14),
    duration: 3,
  },
];

function onChangeColors(newcolors: any) {
  colors.value = newcolors;
}

function onChangeTheme(pallete: any) {
  theming.value = pallete;
}

onMounted(() => window.addEventListener('resize', onWidthChange));
onUnmounted(() => window.removeEventListener('resize', onWidthChange));
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
