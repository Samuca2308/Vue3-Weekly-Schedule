<template>
  <div component type="theme" v-html="theme"></div>
  <main>
    <button @click="showSidebar = !showSidebar" class="btn opt">Menu</button>
    <div
      v-show="showSidebar"
      @click="showSidebar = false"
      class="sidebar-backdrop"
    ></div>
    <aside v-show="windowWidth >= 800 || showSidebar" class="sidebar">
      <button
        @click="showSidebar = false"
        style="text-align: end"
        class="btn opt"
      >
        X
      </button>
      <button @click="showSettings = !showSettings" class="btn accordion">
        Settings
      </button>
      <div
        class="panel"
        :style="`transition: all 200ms ease-in-out; margin-top: 4rem; ${
          showSettings
            ? 'transform: translateY()'
            : 'transform: translateY(calc(-100% - 4rem))'
        }`"
      >
        <Settings
          v-if="showSettings === true"
          :monday="mondayBool"
          :militaryTime="militaryBool"
          @change-opt="onChangeOption"
          @change-colors="onChangeColors"
          @change-theme="onChangeTheme"
        />
      </div>
      <div class="event-form" v-if="curEvent !== null">
        <h2>Event Information</h2>
        <form>
          <label for="description">Event description</label>
          <input
            v-model="curEvent.description"
            name="description"
            type="text"
          />
          <span></span>
        </form>
      </div>
    </aside>
    <WeekSchedule
      :monday="mondayBool"
      :militaryTime="militaryBool"
      :minimal="windowWidth <= 800"
      @event-select="selectEvent"
      size="30rem"
      :records="records"
    />
  </main>
</template>

<script lang="ts" setup>
import WeekSchedule from './components/WeekSchedule.vue';
import Settings from './components/Settings.vue';
import { ref, computed, onMounted, onUnmounted } from 'vue';

const windowWidth = ref(window.innerWidth);
const militaryBool = ref(false);
const mondayBool = ref(false);
const showSidebar = ref(false);
const showSettings = ref(false);
const onWidthChange = () => (windowWidth.value = window.innerWidth);
const curEvent: Ref<Object | null> = ref(null);

const colors = ref(['#de7d14', '#bd357e']);
const theming = ref(['#f5f5f5', '#fff', '#2c3e50']);

const theme = computed(() => {
  return `
    <style>
      :root {
        --primary: ${colors.value[0]};
        --secondary: ${colors.value[1]};
        --canvas: ${theming.value[0]};
        --background: ${theming.value[1]};
        --detail: ${theming.value[2]};
      }
    </style>`;
});

const today = new Date();

const records = [
  {
    description: 'lorem',
    date: new Date(today.getFullYear(), today.getMonth(), today.getDate(), 13),
    duration: 4,
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

function onChangeOption(optNum: any) {
  optNum == 0
    ? (mondayBool.value = !mondayBool.value)
    : (militaryBool.value = !militaryBool.value);
}

function selectEvent(ev: Object) {
  curEvent.value = ev;
}

onMounted(() => window.addEventListener('resize', onWidthChange));
onUnmounted(() => window.removeEventListener('resize', onWidthChange));
</script>

<style>
@keyframes blink {
  to {
    transform: translate(-1.5rem, -1.5rem) scale(4);
    opacity: 0;
  }
}

body {
  margin: 0;
  background-color: var(--canvas);
  color: var(--detail);
}

main {
  display: flex;
}

.event-form h2 {
  margin-block: 0.8rem;
  margin-left: 1.6rem;
}

.event-form label {
  font-size: 0.8rem;
  margin: 0.2rem 2rem;
  position: absolute;
}

.event-form input {
  position: relative;
  width: clamp(6rem, 12%, 18rem);
  background-color: var(--background);
  border: 0.1rem solid var(--detail);
  color: var(--detail);
  border-radius: 0.6rem;
  margin: 1.2rem 2rem;
  padding: 0.2rem 0.4rem;
}

.event-form input:focus {
  outline: none;
  border: 0.1rem solid var(--secondary);
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.sidebar {
  position: relative;
  height: 100vh;
  width: clamp(12rem, 20%, 20rem);
  background-color: var(--background);
  box-shadow: 0 0 0.2rem 0.3rem #00000008;
}

.btn {
  position: absolute;
  z-index: 1;
  cursor: pointer;
  padding: 18px;
  width: 100%;
  text-align: left;
  border: none;
  background-color: var(--background);
  color: var(--detail);
}

.opt {
  display: none;
}

.accordion::before {
  position: absolute;
  content: '';
  top: 0;
  left: 4%;
  width: 92%;
  height: 100%;
  border-bottom: 0.16rem solid var(--primary);
}

@media (max-width: 800px) {
  .sidebar {
    position: fixed;
    z-index: 10;
    top: 0;
  }

  .sidebar-backdrop {
    position: fixed;
    z-index: 10;
    top: 0;
    width: 100%;
    height: 100%;
    background: var(--canvas);
    opacity: 0.4;
  }

  .accordion {
    top: 3.2rem;
  }

  .opt {
    display: block;
  }

  .event-form {
    display: none;
  }
}
</style>
