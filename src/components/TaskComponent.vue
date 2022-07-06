<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, Ref, ref } from 'vue';

const emit = defineEmits(['duration-change', 'event-select']);
const props = defineProps({
  element: Object,
  minimal: {
    type: Boolean,
    default: false,
  },
  militaryTime: {
    type: Boolean,
    default: false,
  },
});

const description = props.element!.description;

const startTime =
  props.element!.date.getHours().toString().padStart(2, '0') +
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

  return h.toString().padStart(2, '0') + ':' + m;
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

function convertTime(time) {
  let hours = Number(time.slice(0, 2));
  let minutes = time.slice(3);

  return (
    (hours == 0 ? 12 : hours > 12 ? Number(hours) - 12 : hours)
      .toString()
      .padStart(2, '0') +
    ':' +
    minutes +
    (hours > 11 ? ' PM' : ' AM')
  );
}

function initObserver() {
  const rem = parseInt(getComputedStyle(document.documentElement).fontSize);

  observer.value = new MutationObserver(function (mutations) {
    mutations.forEach(function (mutation) {
      if (mutation.type === 'attributes') {
        if (resize.value !== true) resize.value = true;
        let height = el.value?.style.height;
        height!.slice(height!.length - 2) !== 'em'
          ? (nDuration.value = Math.round(
              Number(height!.slice(0, height!.length - 2)) / rem / 1.6
            ))
          : (nDuration.value = Math.round(
              Number(height!.slice(0, height!.length - 3)) / 1.6
            ));
      }
    });
  });

  observer.value.observe(el.value!, { attributes: true });
}

function onClick(e: MouseEvent) {
  var elements = document.getElementsByClassName('ripple');
  while (elements[0]) {
    elements[0].parentNode.removeChild(elements[0]);
  }

  emit('event-select');

  let ripple = document.createElement('span');
  ripple.setAttribute('class', 'ripple');
  let bounds = e.target.getBoundingClientRect();

  ripple.style.top = e.clientY - bounds.top + 'px';
  ripple.style.left = e.clientX - bounds.left + 'px';
  el.value!.appendChild(ripple);
}

function onResizeEnd(e: MouseEvent) {
  if (resize.value === true) {
    resize.value = false;
    emit('duration-change', nDuration.value);
  }
}
</script>

<template>
  <article
    @click="onClick"
    @mouseup="onResizeEnd"
    ref="el"
    :style="`height: ${height}rem`"
    :class="
      props.element.flag == 1
        ? 'flag-one'
        : props.element.flag == 2
        ? 'flag-two'
        : ''
    "
  >
    <div v-if="props.minimal != true">
      <p>{{ element.description }}</p>
      <p>
        {{
          `${props.militaryTime ? startTime : convertTime(startTime)} - ${
            props.militaryTime ? endTime : convertTime(endTime)
          }`
        }}
      </p>
    </div>
  </article>
</template>

<style scoped>
::v-global(.ripple) {
  position: absolute;
  border-radius: 50%;
  height: 3rem;
  width: 3rem;
  transform: translate(-1.5rem, -1.5rem) scale(0);
  animation: blink linear 300ms;
  opacity: 0.26;
  background-color: white;
  box-shadow: 0 0 0.6rem 0.2rem var(--secondary);
}

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
  background-color: var(--primary);
}

.flag-one {
  width: 45%;
}

.flag-two {
  background-color: var(--secondary);
  margin-left: calc(50% - 0.1rem);
  width: 40%;
}

.flag-two::after {
  background-color: var(--secondary);
}

article::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: var(--background);
  opacity: 0.56;
  mix-blend-mode: luminosity;
}

article::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  background-color: var(--primary);
  border-radius: 1rem 0 0 1rem;
  width: 0.2rem;
  height: 100%;
}

article > div {
  position: absolute;
}

article p {
  margin-block: 0 0.4rem;
}
</style>
