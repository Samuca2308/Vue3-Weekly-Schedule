<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue';

const emit = defineEmits(['duration-change']);
const props = defineProps({
  element: Object,
});

const description = props.element!.description;
const startTime =
  props.element!.date.getHours() +
  ':' +
  props.element!.date.getMinutes().toString().padStart(2, '0');
const endTime = computed(() => {
  let hours =
    props.element!.date.getHours() +
    Number(props.element!.duration.slice(0, 2));
  let minutes = (
    props.element!.date.getMinutes() + Number(props.element!.duration.slice(3))
  )
    .toString()
    .padStart(2, '0');
  return `${minutes == 60 ? hours + 1 : hours}:${
    minutes == 60 ? '00' : minutes
  }`;
});
const height = computed(() => {
  return (
    props.element!.duration.slice(0, 2) * 3.2 +
    (props.element!.duration.slice(3) / 30) * 1.6 -
    0.4
  );
});
const nDuration = ref('');
const resize = ref(false);
const observer = ref<MutationObserver | null>(null);
const el = ref<HTMLElement | null>(null);

onMounted(() => {
  initObserver();
});

onBeforeUnmount(() => {
  if (observer.value != null) observer.value.disconnect();
});

function initObserver() {
  const rem = parseInt(getComputedStyle(document.documentElement).fontSize);

  observer.value = new MutationObserver(function (mutations) {
    mutations.forEach(function (mutation) {
      if (mutation.type === 'attributes') {
        if (resize.value !== true) resize.value = true;
        let height = el.value?.style.height;
        let minutes =
          Math.round(
            Number(height!.slice(0, height!.length - 2)) / rem / 1.6 + 0.2
          ) * 30;
        nDuration.value =
          Math.floor(minutes / 60)
            .toString()
            .padStart(2, '0') + (minutes % 60 == 30 ? ':30' : ':00');
      }
    });
  });

  observer.value.observe(el.value!, { attributes: true });
}

function func(e: MouseEvent) {
  if (resize.value === true) {
    emit('duration-change', nDuration.value);
    resize.value = false;
  }
}
</script>

<template>
  <article
    @mouseup="func"
    ref="el"
    @resize="resizeEvent"
    :style="`height: ${height}rem`"
  >
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
  left: 0;
  right: 0;
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
