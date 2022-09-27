<template>
  <div class="calendar-display-container">
    <CalendarMonth v-model="date"/>
  </div>
</template>

<script setup>
import CalendarMonth from './components/CalendarMonth.vue';
import { ref } from 'vue'

//const date = ref(new Date(2023, 3, 12));
const date = ref();
</script>

<style scoped>
.calendar-display-container {
  width: 300px;
  margin: 30px auto;
}
</style>
