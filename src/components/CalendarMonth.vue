<template>
  <div class="wrapper">
    <header>
      <p class="current-date">{{theMonthName}} {{theYear}}</p>
      <div class="icons">
        <button type="button" class="prev-month" @click="updateMonth">‹</button>
        <button type="button" class="next-month" @click="updateMonth">›</button>
      </div>
    </header>
    <div class="calendar">
      <ul class="weeks">
        <li>Sun</li>
        <li>Mon</li>
        <li>Tue</li>
        <li>Wed</li>
        <li>Thu</li>
        <li>Fri</li>
        <li>Sat</li>
      </ul>
      <ul class="days">
        <li v-for="day in monthDays" :key="day.day + '-' + day.month" :class="day.type" @click="selectDay(day)">
          {{day.day}}
        </li>        
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

const props = defineProps({
  modelValue: {
    type: Date,
    default: new Date()
  }
});

const emit = defineEmits(['update:modelValue'])

const today = new Date();
let selectedDay = ref(null);
if (props.modelValue) {
  selectedDay = ref(new Date(props.modelValue.getFullYear(), props.modelValue.getMonth(), props.modelValue.getDate()));
}

const monthsList = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
const theYear = ref(props.modelValue.getFullYear());
const theMonth = ref(props.modelValue.getMonth());
const theMonthName = computed(() => {
  return monthsList[theMonth.value];
});

const monthInView = computed(() => {
  return new Date(theYear.value, theMonth.value, 1)
})

function isSameDay(date1, date2) {
  return date1.getDate() === date2.getDate() && date1.getMonth() === date2.getMonth() && date1.getFullYear() === date2.getFullYear();
}

const monthDays = computed(() => {
  const days = [];
  let firstDayOfMonth = ref(new Date(theYear.value, theMonth.value, 1).getDay());
  let lastDateOfMonth = ref(new Date(theYear.value, theMonth.value + 1, 0).getDate());
  let lastDayOfMonth = ref(new Date(theYear.value, theMonth.value, lastDateOfMonth.value).getDay());
  let lastDateOfLastMonth = ref(new Date(theYear.value, theMonth.value, 0).getDate());

  for (let i = firstDayOfMonth.value; i > 0; i--) {
    const month = theMonth.value === 0 ? 12 : theMonth.value - 1;
    const year = theMonth.value === 0 ? theYear.value - 1 : theYear.value;
    const day = lastDateOfLastMonth.value - i + 1;
    let dayClass = 'inactive';
    if (selectedDay.value && isSameDay(new Date(year, month, day), selectedDay.value)) {
      dayClass += ' selected';
    } 

    days.push({
      day: day,
      month: month,
      type: dayClass,
    })
  }

  for (let i = 1; i <= lastDateOfMonth.value; i++) {
    const isToday = i === today.getDate() && theMonth.value === today.getMonth()
                    && theYear.value === today.getFullYear();
    let dayClass = 'active';
    if (isToday) {
      dayClass += ' current';
    }
    if (selectedDay.value && isSameDay(new Date(theYear.value, theMonth.value, i), selectedDay.value)) {
      dayClass += ' selected';
    } 
    days.push({
      day: i,
      month: theMonth.value,
      type: dayClass,
    })
  }

  for (let i = lastDayOfMonth.value; i < 6; i++) {
    let dayClass = 'inactive';
    const month = theMonth.value === 11 ? 0 : theMonth.value + 1;
    const year = theMonth.value === 11 ? theYear.value + 1 : theYear.value;
    const day = i - lastDayOfMonth.value + 1;
    if (selectedDay.value && isSameDay(new Date(year, month, day), selectedDay.value)) {
      dayClass += ' selected';
    } 
    days.push({
      day: day,
      month: month,
      type: dayClass,
    })
  }

  return days;
});


function updateMonth(event) {
  const isPrev = event.target.classList.contains('prev-month');
  const monthToBeDisplayed = isPrev ? theMonth.value - 1 : theMonth.value + 1;

  if (monthToBeDisplayed < 0 || monthToBeDisplayed > 11) {
    const newDate = new Date(theYear.value, monthToBeDisplayed);
    theMonth.value = newDate.getMonth();
    theYear.value = newDate.getFullYear();
  } else {
    if (isPrev) {
      theMonth.value = theMonth.value - 1;
    } else {
      theMonth.value = theMonth.value + 1;
    }
  } 
}

function selectDay(day) {
  selectedDay.value = new Date(theYear.value, day.month, day.day);
}

watch(selectedDay, (newSelectedDay) => {
  emit('update:modelValue', newSelectedDay)
})

</script>

<style scoped>
.wrapper {
  background: #ffffff;
  width: 450px;
  border-radius: 10px;
}

.wrapper header{
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 25px 30px 10px;
}

header .current-date {
  font-size: 1.45rem;
  font-weight: 500;
}

header .icons button {
  height: 38px;
  width: 38px;
  margin: 0 1px;
  color: #878787;
  line-height: 36px;
  border-radius: 50%;
  text-align: center;
  cursor: pointer;
  font-size: 1.9rem;
  display: inline-block;
  outline: none;
  border: none;
}

header .icons button:hover {
  background-color: #f2f2f2;
}

header .icons button:last-child {
  margin-right: -10px;
}

.calendar {
  padding: 20px;
}

.calendar ul {
  display: flex;
  list-style: none;
  flex-wrap: wrap;
  text-align: center;
} 

.calendar .days {
  margin-bottom: 20px;
}

.calendar .weeks li {
  font-weight: 500;
}

.calendar .days li {
  cursor: pointer;
  margin-top: 30px;
  z-index: 1; 
}

.calendar ul.days li:before {
  position: absolute;
  content: '';
  height: 40px;
  width: 40px;
  top: 50%;
  left: 50%;
  z-index: -1;
  border-radius: 50%;
  transform: translate(-50%, -50%); 
}

.days li:hover::before {
  background-color: #f2f2f2;
}

.days li.inactive {
  color: #aaaaaa;
}

.days li.current {
  color: #333333;
}

.days li.selected {
  color: #ffffff;
}

.days li.current::before {
  background-color: #ffffff;
  border: 1px solid #aaaaaa;  
}

.days li.selected::before {
  background-color: #9B59B6;  
}

.calendar ul li {
  position: relative;
  width: calc(100% / 7);
}
</style>
