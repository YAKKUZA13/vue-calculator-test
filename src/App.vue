<script setup lang="ts">
import CalculatorForm from './components/CalculatorForm.vue'
</script>

<template>
  <div id="app">
    <div class="background-shapes">
      <div class="shape shape-1"></div>
      <div class="shape shape-2"></div>
      <div class="shape shape-3"></div>
    </div>
    <header class="app-header">
      <h1>
        <span class="icon">🧮</span>
        Vue Calculator
        <span class="subtitle">Modern Test Application</span>
      </h1>
    </header>
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
  font-family: 'Inter', 'SF Pro Display', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: #0a0a0a;
  overflow-x: hidden;
}

#app {
  min-height: 100vh;
  width: 100%;
  position: relative;
  background: linear-gradient(135deg, 
    #667eea 0%, 
    #764ba2 25%, 
    #f093fb 50%, 
    #f5576c 75%, 
    #4facfe 100%);
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
  display: flex;
  flex-direction: column;
  padding: 0;
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.background-shapes {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  pointer-events: none;
}

.shape {
  position: absolute;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 50%;
  filter: blur(2px);
  animation: float 20s infinite ease-in-out;
}

.shape-1 {
  width: 300px;
  height: 300px;
  top: 20%;
  left: 10%;
  animation-delay: 0s;
}

.shape-2 {
  width: 200px;
  height: 200px;
  top: 60%;
  right: 15%;
  animation-delay: 7s;
}

.shape-3 {
  width: 150px;
  height: 150px;
  bottom: 20%;
  left: 60%;
  animation-delay: 14s;
}

@keyframes float {
  0%, 100% { transform: translateY(0px) rotate(0deg); }
  33% { transform: translateY(-30px) rotate(120deg); }
  66% { transform: translateY(30px) rotate(240deg); }
}

.app-header {
  position: relative;
  z-index: 10;
  text-align: center;
  padding: 40px 20px 20px;
}

h1 {
  color: #ffffff;
  font-size: clamp(2rem, 5vw, 3.5rem);
  font-weight: 800;
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  letter-spacing: -0.02em;
  margin-bottom: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.icon {
  font-size: 1.2em;
  display: block;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-10px); }
  60% { transform: translateY(-5px); }
}

.subtitle {
  font-size: 0.4em;
  font-weight: 400;
  opacity: 0.9;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  margin-top: 5px;
}

.app-main {
  flex: 1;
  position: relative;
  z-index: 10;
  padding: 0 20px 40px;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}

/* Responsive */
@media (max-width: 768px) {
  .app-header {
    padding: 20px 15px 15px;
  }
  
  .app-main {
    padding: 0 15px 20px;
  }
  
  .shape {
    display: none;
  }
}
</style>
