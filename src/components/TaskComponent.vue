<script setup lang="ts">
import { ref } from 'vue';

const props = defineProps({
  element: Object,
});

const description = props.element!.description;
const startTime =
  props.element!.date.getHours() +
  ':' +
  props.element!.date.getMinutes().toString().padStart(2, '0');
const endTime =
  props.element!.date.getHours() +
  Number(props.element!.duration.slice(0, 2)) +
  ':' +
  (props.element!.date.getMinutes() + Number(props.element!.duration.slice(3)))
    .toString()
    .padStart(2, '0');
const height = ref(
  props.element!.duration.slice(0, 2) * 3.2 +
    (props.element!.duration.slice(3) / 30) * 1.6 -
    0.4
);

function resizeEvent() {
  console.log('e');
}
</script>

<template>
  <article @resize="resizeEvent" :style="`height: ${height}rem`">
    <p>{{ element.description }}</p>
    <p v-if="element.duration.slice(0, 2) > 0">
      {{ `${startTime} - ${endTime}` }}
    </p>
  </article>
</template>

<style scoped>
article {
  cursor: pointer;
  padding: 0.2rem 0.4rem;
  z-index: 1;
  resize: vertical;
  overflow: hidden;
  position: absolute;
  margin-inline: 0.1rem;
  top: 0;
  border-radius: 0.2rem;
  width: stretch;
  background-color: var(--primary-light);
}
article::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  background-color: var(--primary);
  border-radius: 1rem 0 0 1rem;
  width: 0.2rem;
  height: 100%;
}
article > p {
  margin-block: 0 0.4rem;
}
</style>
