<template>
  <h3 v-if="date">Date is {{date.toLocaleDateString()}}</h3>
  <CalendarMonth v-model="date"/>
</template>

<script setup>
import CalendarMonth from './components/CalendarMonth.vue';
import { ref } from 'vue'

const date = ref(new Date(2023, 3, 12));
//const date = ref();
</script>

<style scoped>

</style>
