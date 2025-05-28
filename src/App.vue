<script setup lang="ts">
import CalculatorForm from './components/CalculatorForm.vue'
</script>

<template>
  <div id="app">
 
    <CalculatorForm />
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
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f0f2f5;
}

#app {
  min-height: 100vh;
  width: 100%;
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
}

h1 {
  text-align: center;
  color: #ffffff;
  margin-bottom: 30px;
  font-size: clamp(1.8rem, 4vw, 2.5rem);
  font-weight: 700;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  letter-spacing: 2px;
  text-transform: uppercase;
  width: 100%;
}
</style>
