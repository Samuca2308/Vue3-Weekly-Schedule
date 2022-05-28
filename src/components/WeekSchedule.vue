<script setup lang="ts">
import { computed, onMounted, Ref, ref } from 'vue';
import taskComponent from './TaskComponent.vue';

const props = defineProps({
  records: Array,
  date: {
    type: Date,
    default: new Date(),
  },
  size: {
    type: String,
    default: '26rem',
  },
  weekArr: {
    type: Array,
    default: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
  },
  militaryTime: {
    type: Boolean,
    default: false,
  },
  monday: {
    type: Boolean,
    default: false,
  },
});

const curRecords: Ref<any> = ref([]);

const week = computed(() => {
  const curDate = props.date.getDate();
  const curDay = props.date.getDay();
  const forMonthDays = new Date(
    props.date.getFullYear(),
    props.date.getMonth(),
    0
  ).getDate();
  const curMonthDays = new Date(
    props.date.getFullYear(),
    props.date.getMonth() + 1,
    0
  ).getDate();

  let arr = [];
  for (let i = 0; i < 7; i++) {
    props.monday === true
      ? arr.push({
          name: i == 6 ? props.weekArr[0] : props.weekArr[i + 1],
          number: () => {
            let day =
              curDay == 0 ? curDate + i - 6 : curDate - (curDay - 1) + i;
            return day < 1
              ? (day = day + forMonthDays)
              : day > curMonthDays
              ? (day = day - curMonthDays)
              : day;
          },
          today: (i == 6 && curDay == 0) || i + 1 == curDay,
        })
      : arr.push({
          name: props.weekArr[i],
          number: () => {
            let day = curDate - curDay + i;
            return day < 1
              ? (day = day + forMonthDays)
              : day > curMonthDays
              ? (day = day - curMonthDays)
              : day;
          },
          today: i - curDay == 0,
        });
  }
  return arr;
});

const hours = computed(() => {
  let arr = [];
  if (props.militaryTime === true) {
    for (let x = 0; x < 24; x++) {
      arr.push(`${x.toString().padStart(2, '0')}:00`);
    }
  } else {
    for (let x = 0; x < 24; x++) {
      arr.push(
        `${
          x == 0 || x == 12
            ? '12'
            : x < 12
            ? x.toString().padStart(2, '0')
            : (x - 12).toString().padStart(2, '0')
        }:00 ${x < 12 ? 'AM' : 'PM'}`
      );
    }
  }
  return arr;
});

function startArr() {
  const startDate = new Date(
    `${props.date.getFullYear()}/${
      props.date.getMonth() + 1
    }/${week.value[0].number()}`
  );
  const endDate = new Date(
    `${props.date.getFullYear()}/${props.date.getMonth() + 1}/${
      week.value[week.value.length - 1].number() + 1
    }`
  );
  curRecords.value = props.records
    ?.filter((rec: any) => {
      return rec.date >= startDate && rec.date <= endDate;
    })
    .reduce(
      (pvel: any, el: any) => ({
        ...pvel,
        [`${
          props.monday === true
            ? el.date.getDay() == 0
              ? 6
              : el.date.getDay() - 1
            : el.date.getDay()
        }${(el.date.getMinutes() >= 30
          ? 1 + el.date.getHours() * 2
          : el.date.getHours() * 2
        )
          .toString()
          .padStart(2, '0')}`]: el,
      }),
      {}
    );
}

onMounted(() => {
  startArr();
});

function dragEvent(e: any, id: any) {
  e.dataTransfer!.dropEffect = 'move';
  e.dataTransfer!.effectAllowed = 'move';
  e.dataTransfer!.setData('eventValue', id);
}

function dropEvent(e: any, dayId: any, hourId: any) {
  const value = e.dataTransfer!.getData('eventValue');
  if (value) {
    const item: any = curRecords.value![value];
    item!.date = new Date(
      `${props.date.getFullYear()}/${props.date.getMonth() + 1}/${week.value[
        dayId - 1
      ].number()}, ${Math.ceil(hourId / 2) - 1}:${
        hourId % 2 == 0 ? '30' : '00'
      }`
    );
    startArr();
  }
}

function changeEvent(e: any, id: string) {
  /* WIP Fix */
  curRecords.value![id].duration = e;
}
</script>

<template>
  <table>
    <thead>
      <tr>
        <th></th>
        <th
          :key="week.indexOf(day)"
          v-for="day in week"
          :class="day.today ? 'today' : ''"
        >
          <h3>
            {{ day.name }}
          </h3>
          <h3>
            {{ day.number() }}
          </h3>
        </th>
      </tr>
    </thead>
    <tbody :style="`height: ${props.size};`">
      <tr :key="numH" v-for="numH in 48">
        <td class="hour">
          {{ numH % 2 == 1 ? hours[Math.ceil(numH / 2) - 1] : '' }}
        </td>
        <td
          :key="numD"
          v-for="numD in 7"
          @drop.prevent="dropEvent($event, numD, numH)"
          @dragenter.prevent
          @dragover.prevent
        >
          <component
            draggable="true"
            @dragstart="
              dragEvent(
                $event,
                (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
              )
            "
            @duration-change="
              changeEvent(
                $event,
                (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
              )
            "
            v-if="
              curRecords[
                (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
              ]
            "
            v-bind="{
              element:
                curRecords[
                  (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
                ],
            }"
            :is="taskComponent"
          ></component>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped>
.today {
  color: var(--primary);
}
.hour {
  width: 3.6rem;
  font-size: 0.84rem;
  padding: 0.2rem;
}
table {
  margin: 2rem auto;
  padding: 0.4rem 1.2rem;
  border: 1px solid var(--detail);
  border-radius: 0.8rem;
  border-spacing: 0;
  width: clamp(26rem, 70%, 96rem);
}
table thead,
table tbody tr {
  display: table;
  table-layout: fixed;
  width: 100%;
}
table thead th {
  cursor: pointer;
}
table thead th:first-of-type {
  cursor: default;
  width: 4rem;
}
table thead th h3 {
  margin: 0;
}
table thead th h3:last-child {
  font-size: 1.8rem;
}
table tbody {
  display: block;
  padding: 1.2rem 0;
  overflow-y: auto;
}
table tbody tr {
  --primary-border: 0.01rem solid #56565676;
  --secondary-border: 0.01rem solid #56565636;
  height: 1.6rem;
}
table tbody tr td {
  padding: 0;
  user-select: none;
  position: relative;
}
table tbody tr td:nth-child(2) {
  border-left: var(--primary-border);
}
table tbody tr td:not(:first-child) {
  border-top: var(--primary-border);
  border-right: var(--primary-border);
}
table tbody tr:nth-child(2n) td:not(:first-child) {
  border-top: var(--secondary-border);
}
table tbody tr:last-child td:not(:first-child) {
  border-bottom: var(--primary-border);
}
</style>
