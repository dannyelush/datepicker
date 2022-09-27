<template>
  <div ref="el">
    <input type="text" @click="show = true" v-model="dateDisplay" class="date-input"
      placeholder="Select date"/>
    <div class="wrapper" v-if="show" ref="dropdown">    
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
  </div>
</template>

<script setup>
import { ref, computed, watch, nextTick } from 'vue';

const props = defineProps({
  modelValue: {
    type: Date,
    default: new Date()
  },
  closeOnEsc: {
    type: Boolean,
    default: true,
  },
  displayToday: {
    type: Boolean,
    default: true,
  }
});

const emit = defineEmits(['update:modelValue']);

let show = ref(false);

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
    const isToday = props.displayToday && i === today.getDate() && theMonth.value === today.getMonth()
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
  show.value = false;
  emit('update:modelValue', newSelectedDay);
});

const dateDisplay = computed(() => {
  return new Intl.DateTimeFormat('en-US').format(selectedDay.value);
})

// Logic for clicking inside and outside of calendar
const dropdown = ref(null);
const el = ref(null);

function cancel() {
  show.value = false;
}

function clickAway($event) {
  if ($event && $event.target && !el.value.contains($event.target) 
      && dropdown && !dropdown.value.contains($event.target)) {
    cancel();
  }
}

function handleEscape(e) {
  if (open && e.keyCode === 27 && props.closeOnEsc) {
    cancel();
    el.value.blur();
  }
}

watch(show, async (newShow) => {
  if (typeof document === "object") {
    await nextTick();

    newShow ? document.body.addEventListener('click', clickAway) : document.body.removeEventListener('click', clickAway);
    newShow ? document.addEventListener('keydown', handleEscape) : document.removeEventListener('keydown', handleEscape)
  }
});

</script>

<style scoped>
.wrapper {
  background: #ffffff;
  width: 300px;
  border-radius: 5px;
  margin-top: 1px;
}

.wrapper header{
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 25px 20px 0;
}

header .current-date {
  font-size: 1rem;
  font-weight: 500;
}

header .icons button {
  height: 30px;
  width: 30px;
  margin: 0 1px;
  color: #878787;
  line-height: 28px;
  border-radius: 50%;
  text-align: center;
  cursor: pointer;
  font-size: 1.5rem;
  display: inline-block;
  outline: none;
  border: none;
}

header .icons button:hover {
  background-color: #f2f2f2;
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
  margin-bottom: 10px;
}

.calendar .weeks li {
  font-weight: 500;
  font-size: 14px;
}

.calendar .days li {
  cursor: pointer;
  margin-top: 25px;
  z-index: 1; 
  font-size: 12px;
}

.calendar ul.days li:before {
  position: absolute;
  content: '';
  height: 30px;
  width: 30px;
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

.days li.current:not(.selected)::before {
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

.date-input {
  width: 100%;
  height: 30px;
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
}
</style>
