<script lang="ts" setup>
import { ref } from 'vue';
const emit = defineEmits(['change-theme', 'change-colors']);

const colorsArr = [
  { name: 'Orange', colors: ['#de7d14', '#de7d1472', '#bd357e', '#bd357e72'] },
  { name: 'Lime', colors: ['#81d022', '#81d02272', '#d89730', '#d8973072'] },
  { name: 'Red', colors: ['#9b0b16', '#9b0b1672', '#837172', '#83717272'] },
  { name: 'Blue', colors: ['#1a868e', '#1a868e72', '#72704a', '#72704a72'] },
];

const themes = {
  light: ['#fff', '#2c3e50'],
  dark: ['#000', '#d3c1af'],
};

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

function changeTheme(theme: any) {
  emit('change-theme', theme);
}

function changeColor(colors: any) {
  emit('change-colors', colors);
}
</script>

<template>
  <nav>
    <a>Schedule</a>
    <a>Info</a>
    <a @click="toggleModal">Theme</a>
  </nav>
  <dialog id="color-picker" class="color-picker">
    <h3>Theme</h3>
    <div>
      <a @click="changeTheme(themes.dark)"> Dark Theme </a>
      <a @click="changeTheme(themes.light)"> Bright Theme </a>
    </div>
    <h3>Colors</h3>
    <div>
      <a
        :key="colorsArr.indexOf(color)"
        v-for="color in colorsArr"
        @click="changeColor(color.colors)"
        :style="`--pri: ${color.colors[0]}; --sec: ${color.colors[2]};`"
      >
        {{ color.name }}
      </a>
    </div>
  </dialog>
</template>

<style scoped>
nav {
  height: 3.4rem;
  background: var(--primary);
  filter: saturate(50%);
  display: flex;
  justify-content: space-evenly;
}
nav > a {
  color: black;
  margin: auto 0;
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 300ms ease-in-out;
}
nav > a:hover,
nav > a:focus {
  color: white;
}
.color-picker {
  width: 21rem;
  border: none;
  border-radius: 0.6rem;
  padding: 0.5rem 1.2rem;
  background: var(--background);
  color: var(--text-color);
}
.color-picker h3 {
  margin: 0;
}
.color-picker div {
  display: flex;
  justify-content: space-around;
}
.color-picker div:first-of-type a::before {
  border: 0.01rem solid #464646 58;
}
.color-picker div a {
  cursor: pointer;
  margin-left: 2rem;
  margin-block: 1.2rem;
  position: relative;
}
.color-picker div a::before {
  content: '';
  position: absolute;
  height: 1.4rem;
  width: 1.4rem;
  border-radius: 0.7rem;
  background-color: black;
  top: -0.1rem;
  left: -1.8rem;
}
.color-picker div a:last-child::before {
  background-color: white;
}
.color-picker div:last-child a {
  cursor: pointer;
  position: relative;
}
.color-picker div:last-child a::before {
  height: 1.4rem;
  width: 1.4rem;
  border-radius: 50%;
  background-color: var(--sec);
  top: 0;
  left: -1.8rem;
}
.color-picker div:last-child a::after {
  content: '';
  position: absolute;
  height: 0.86rem;
  width: 0.86rem;
  border-radius: 50%;
  background-color: var(--pri);
  border: 0.12rem solid var(--background);
  top: 0;
  left: -1.48rem;
}
</style>
