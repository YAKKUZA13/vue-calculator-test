<script setup lang="ts">
import CalculatorForm from './components/CalculatorForm.vue'
</script>

<template>
  <div id="app">
    <main class="app-main">
      <CalculatorForm />
    </main>
  </div>
</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
  background: #f5f5f5;
}

#app {
  min-height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  padding: 20px;
}

.app-main {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 20px;
}

@media (max-width: 768px) {
  .app-main {
    padding: 10px;
  }
}
</style>
