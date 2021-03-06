<script setup lang="ts">
import { computed, onMounted, Ref, ref, watch } from 'vue';
import taskComponent from './TaskComponent.vue';

const emit = defineEmits(['event-select']);
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
  minimal: {
    type: Boolean,
    default: false,
  },
});

watch(
  () => props.monday,
  () => {
    refreshArr(curRecords.value!);
  }
);

const year = ref(props.date.getFullYear());
const month = ref(props.date.getMonth() + 1);
const monthProg = ref(1);

const sortedArr: Ref<any> = ref([]);
const curRecords: Ref<any> = ref([]);

const dropMode = ref(false);

/**
 * Creates an array for the week header,
 * consequently specifying values for the days being used in the current week.
 * Later to be referenced in refreshArr for filtering registers.
 */
const week = computed(() => {
  const curDate = props.date.getDate();
  const curDay = props.date.getDay();
  const forMonthDays = new Date(year.value, month.value - 1, 0).getDate();
  const curMonthDays = new Date(year.value, month.value, 0).getDate();

  let arr = [];
  for (let i = 0; i < 7; i++) {
    props.monday === true
      ? arr.push({
          name: i == 6 ? props.weekArr[0] : props.weekArr[i + 1],
          number: () => {
            let day = extrCalc(
              curDay == 0 ? curDate + i - 6 : curDate - (curDay - 1) + i,
              curMonthDays,
              forMonthDays
            );
            return day;
          },
          today: (i == 6 && curDay == 0) || i + 1 == curDay,
        })
      : arr.push({
          name: props.weekArr[i],
          number: () => {
            let day = extrCalc(
              curDate - curDay + i,
              curMonthDays,
              forMonthDays
            );
            return day;
          },
          today: i - curDay == 0,
        });
  }
  return arr;
});

// Used by week for returning day numbers and month progress.
function extrCalc(cur: any, min: any, max: any) {
  if (cur < 1) {
    cur = cur + max;
    monthProg.value = 0;
  } else if (cur > min) {
    cur = cur - min;
    monthProg.value = 2;
  }

  return cur;
}

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

const limits = computed(() => {
  let initDay = week.value[0].number();
  let finalDay = week.value[week.value.length - 1].number() + 1;

  const startDate = new Date(
    `${year.value}/${
      finalDay < initDay && monthProg.value == 0 ? month.value - 1 : month.value
    }/${initDay}`
  );
  const endDate = new Date(
    `${year.value}/${
      finalDay < initDay && monthProg.value == 2 ? month.value + 1 : month.value
    }/${finalDay}`
  );

  return { initial: startDate, final: endDate };
});

function refreshArr(sourceArr: any) {
  sourceArr.forEach((object) => {
    delete object['cell'];
    delete object['flag'];
  });
  sortedArr.value = sourceArr
    .filter((rec: any) => {
      return rec.date >= limits.value.initial && rec.date <= limits.value.final;
    })
    .sort((a, b) => a.date - b.date)
    .map((el) => {
      let cellId = calcCellId(el.date);
      return { cell: cellId, ...el };
    });
  sortedArr.value.forEach((el) => {
    let cellId = calcCellId(el.date);
    for (let i = 0; i < el.duration; i++) {
      let concurrent = sortedArr.value.find(
        (conEl) => conEl != el && conEl.cell == Number(cellId) + i
      );
      if (concurrent != null) {
        let conId = sortedArr.value.indexOf(concurrent);
        sortedArr.value[conId] = {
          ...concurrent,
          flag: concurrent.flag ? concurrent.flag + 1 : 2,
        };
        sortedArr.value[sortedArr.value.indexOf(el)] = {
          ...el,
          flag: el.flag ? el.flag + 1 : 1,
        };
      }
    }
  });
  curRecords.value = sortedArr.value;
}

function calcCellId(date) {
  let daySection =
    props.monday === true
      ? date.getDay() == 0
        ? 6
        : date.getDay() - 1
      : date.getDay();
  let hourSection =
    date.getMinutes() >= 30 ? 1 + date.getHours() * 2 : date.getHours() * 2;
  return `${daySection}${hourSection.toString().padStart(2, '0')}`;
}

onMounted(() => {
  refreshArr(props.records);
});

function dragEvent(e: any, id: any) {
  e.dataTransfer!.dropEffect = 'move';
  e.dataTransfer!.effectAllowed = 'move';
  e.dataTransfer!.setData('eventValue', id);
  setTimeout(() => {
    dropMode.value = true;
  }, 2);
}

function dropEvent(e: any, dayId: any, hourId: any) {
  const value = e.dataTransfer!.getData('eventValue');

  if (value != null && value != undefined) {
    const item: any = sortedArr.value!.find((el) => el.cell == value);
    item!.date = new Date(
      `${year.value}/${
        monthProg.value == 2 && week.value[dayId - 1].number() < 7
          ? month.value + 1
          : monthProg.value == 0 && week.value[dayId - 1].number() > 21
          ? month.value - 1
          : month.value
      }/${week.value[dayId - 1].number()}, ${Math.ceil(hourId / 2) - 1}:${
        hourId % 2 == 0 ? '30' : '00'
      }`
    );

    refreshArr(curRecords.value!);
  }
}

function changeEvent(e: any, id: string) {
  curRecords.value!.find((el) => el.cell == id).duration = e;
}

function selectEvent(id: string) {
  emit(
    'event-select',
    curRecords.value!.find((el) => el.cell == id)
  );
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
        <td :key="numD" v-for="numD in 7">
          <component
            draggable="true"
            @dragstart="
              dragEvent(
                $event,
                (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
              )
            "
            @dragend="dropMode = false"
            @duration-change="
              changeEvent(
                $event,
                (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
              )
            "
            @event-select="
              selectEvent(
                (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
              )
            "
            v-if="
              curRecords.find(
                (el) =>
                  el.cell ==
                  (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
              ) != null
            "
            v-bind="{
              element: curRecords.find(
                (el) =>
                  el.cell ==
                  (numD - 1).toString() + (numH - 1).toString().padStart(2, '0')
              ),
              militaryTime: props.militaryTime,
              minimal: props.minimal,
            }"
            :is="taskComponent"
          ></component>
          <div
            class="drop-area"
            v-if="dropMode == true"
            @drop.prevent="dropEvent($event, numD, numH)"
            @dragenter.prevent
            @dragover.prevent
          ></div>
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
  text-align: end;
  font-size: 0.84rem;
  padding-inline: 0.2rem;
}

.drop-area {
  position: absolute;
  transform: translateY(-50%);
  width: 100%;
  height: 100%;
  z-index: 1;
}

table {
  margin: 4rem auto;
  height: min-content;
  padding: 0.4rem 1.2rem;
  border-radius: 0.8rem;
  border-spacing: 0;
  background-color: var(--background);
  box-shadow: 0 0.2rem 0.2rem 0.1rem #00000008;
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

@media (max-width: 800px) {
  table {
    padding: 0.4rem 0.2rem;
  }
  tbody tr {
    position: relative;
  }
  .hour {
    position: absolute;
    top: 0.2rem;
    right: 100%;
    z-index: 1;
    transform: translateX(100%);
  }
}
</style>
