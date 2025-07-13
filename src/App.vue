<script setup lang="ts">
import Calendar from './components/Calendar.vue'
</script>

<template>
  <div id="app">
    <header class="app-header">
      <h1>📅 Vue Calendar App</h1>
      <p>Drag and drop to create events, click to edit</p>
    </header>
    <main>
      <Calendar />
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f7fa;
  color: #333;
}

#app {
  min-height: 100vh;
  padding: 20px;
}

.app-header {
  text-align: center;
  margin-bottom: 30px;
}

.app-header h1 {
  color: #2c3e50;
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.app-header p {
  color: #666;
  font-size: 1.1rem;
}
</style>
