<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, Ref, ref } from 'vue';

const emit = defineEmits(['duration-change']);
const props = defineProps({
  element: Object,
  minimal: {
    type: Boolean,
    default: false,
  },
});

const description = props.element!.description;
const startTime =
  props.element!.date.getHours() +
  ':' +
  props.element!.date.getMinutes().toString().padStart(2, '0');
const endTime = computed(() => {
  let forM = props.element!.date.getMinutes();
  let h =
    props.element!.date.getHours() + Math.floor(props.element!.duration / 2);
  let m = props.element!.duration % 2 == 1 ? '30' : '00';
  if (forM >= 30) {
    m = props.element!.duration % 2 == 1 ? '00' : '30';
    h = m == '30' ? h : h + 1;
  }

  return h + ':' + m;
});
const height = computed(() => {
  return props.element!.duration * 1.6 - 0.6;
});
const nDuration: Ref<number | null> = ref(null);
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
        nDuration.value = Math.round(
          Number(height!.slice(0, height!.length - 2)) / rem / 1.6
        );
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
    <div v-if="props.minimal != true">
      <p>{{ element.description }}</p>
      <p>{{ `${startTime} - ${endTime}` }}</p>
    </div>
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
  top: 0.2rem;
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
article p {
  margin-block: 0 0.4rem;
}
</style>
