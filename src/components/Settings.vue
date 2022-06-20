<script lang="ts" setup>
import { ref } from 'vue';
const emit = defineEmits(['change-theme', 'change-colors', 'change-opt']);

const props = defineProps({
  militaryTime: {
    type: Boolean,
    required: true,
  },
  monday: {
    type: Boolean,
    required: true,
  },
});

const colorsArr = [
  { colors: ['#de7d14', '#bd357e'] },
  { colors: ['#81d022', '#d89730'] },
  { colors: ['#9b0b16', '#837172'] },
  { colors: ['#1a868e', '#72704a'] },
];

const themes = {
  light: ['#f5f5f5', '#fff', '#2c3e50'],
  dark: ['#000', '#101010', '#fff'],
};

const lightTheme = ref(true);

function toggleModal() {
  const modal: any = document.getElementById('color-picker');
  modal!.showModal();
  modal!.addEventListener(
    'click',
    function (event: { clientY: number; clientX: number }) {
      var rect = modal!.getBoundingClientRect();
      var isInDialog =
        rect.top <= event.clientY &&
        event.clientY <= rect.top + rect.height &&
        rect.left <= event.clientX &&
        event.clientX <= rect.left + rect.width;
      if (!isInDialog) {
        modal!.close();
      }
    }
  );
}

function changeTheme(theme: any, isLight: any) {
  lightTheme.value = isLight;
  emit('change-theme', theme);
}

function changeColor(colors: any) {
  emit('change-colors', colors);
}

function changeOption(optNum: any) {
  emit('change-opt', optNum);
}
</script>

<template>
  <article id="color-picker" class="color-picker">
    <div>
      <h3>Theme</h3>
      <div class="theme-selector">
        <a
          :class="lightTheme ? '' : 'selected-theme'"
          style="background-color: black"
          @click="changeTheme(themes.dark, false)"
        ></a>
        <a
          :class="lightTheme ? 'selected-theme' : ''"
          style="background-color: white"
          @click="changeTheme(themes.light, true)"
        >
        </a>
      </div>
    </div>
    <h3 style="margin-top: 1rem">Colors</h3>
    <div class="color-selector">
      <a
        :key="colorsArr.indexOf(color)"
        v-for="color in colorsArr"
        @click="changeColor(color.colors)"
        :style="`--sec: ${color.colors[1]}; background-color: ${color.colors[0]}`"
      >
      </a>
    </div>
    <h3 style="margin-block: 1rem">Options</h3>
    <div>
      <a
        @click="changeOption(0)"
        :class="`checkbox ${props.monday ? 'checked' : ''}`"
        >start on monday</a
      >
      <a
        @click="changeOption(1)"
        :class="`checkbox ${props.militaryTime ? 'checked' : ''}`"
        >military time</a
      >
    </div>
  </article>
</template>

<style scoped>
.color-picker {
  padding: 0.5rem 1.2rem;
}

.selected-theme {
  box-shadow: 0 0 0 0.1rem var(--background), 0 0 0 0.2rem var(--secondary);
}

.color-picker h3 {
  margin: 0;
  padding: 0.2rem;
}

.color-picker h3::before {
  content: '▸';
  margin-right: 0.4rem;
  color: var(--primary);
}

.color-picker div {
  display: flex;
  justify-content: space-between;
}

.theme-selector a {
  cursor: pointer;
  margin-left: 0.6rem;
  margin-block: 0.2rem;
  height: 1.2rem;
  width: 1.2rem;
  border-radius: 50%;
}

.color-selector a {
  cursor: pointer;
  position: relative;
  margin: 1rem 0 0 0.6rem;
  overflow: hidden;
  height: 1.2rem;
  width: 1.2rem;
  border-radius: 50%;
}

.color-selector a::after {
  content: '';
  position: absolute;
  top: 0.09rem;
  left: 0.25rem;
  height: 0.6rem;
  width: 1.2rem;
  transform: rotate(50deg);
  background-color: var(--sec);
}

.checkbox {
  position: relative;
  cursor: pointer;
  margin: 0 2rem;
}

.checkbox::before {
  position: absolute;
  content: '';
  left: -1rem;
  top: 0.15rem;
  width: 0.7rem;
  height: 0.7rem;
  border: 1px solid var(--detail);
  border-radius: 0.3rem;
  opacity: 0.32;
}

.checked::after {
  position: absolute;
  color: var(--secondary);
  font-size: 1.2rem;
  content: '✔';
  left: -1.1rem;
  top: -0.3rem;
}
</style>
