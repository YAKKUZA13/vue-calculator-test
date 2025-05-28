<script setup lang="ts">
import CalculatorForm from './components/CalculatorForm.vue'
</script>

<template>
  <div id="app">
    <div class="background-grid">
      <div class="grid-line"></div>
      <div class="grid-line"></div>
      <div class="grid-line"></div>
      <div class="grid-line"></div>
    </div>
    <div class="background-shapes">
      <div class="shape shape-1"></div>
      <div class="shape shape-2"></div>
      <div class="shape shape-3"></div>
    </div>
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
  font-family: 'Rajdhani', 'Orbitron', 'Roboto Condensed', sans-serif;
  background: #0a0a0a;
  overflow-x: hidden;
}

#app {
  min-height: 100vh;
  width: 100%;
  position: relative;
  background: linear-gradient(135deg, 
    #1a1a1a 0%, 
    #2d2d2d 25%, 
    #1f1f1f 50%, 
    #333333 75%, 
    #0f0f0f 100%);
  background-size: 400% 400%;
  animation: darkShift 20s ease infinite;
  display: flex;
  flex-direction: column;
  padding: 0;
}

@keyframes darkShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.background-grid {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  pointer-events: none;
  opacity: 0.1;
}

.grid-line {
  position: absolute;
  background: linear-gradient(90deg, 
    transparent, 
    #ff6b35, 
    #f7931e, 
    transparent);
  height: 1px;
  width: 100%;
  animation: gridPulse 4s infinite ease-in-out;
}

.grid-line:nth-child(1) { top: 25%; animation-delay: 0s; }
.grid-line:nth-child(2) { top: 50%; animation-delay: 1s; }
.grid-line:nth-child(3) { top: 75%; animation-delay: 2s; }
.grid-line:nth-child(4) { 
  top: 0; 
  left: 50%; 
  width: 1px; 
  height: 100%; 
  animation-delay: 3s;
  background: linear-gradient(180deg, 
    transparent, 
    #ff6b35, 
    #f7931e, 
    transparent);
}

@keyframes gridPulse {
  0%, 100% { opacity: 0.1; }
  50% { opacity: 0.3; }
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
  background: rgba(255, 107, 53, 0.05);
  border: 1px solid rgba(255, 107, 53, 0.1);
  animation: geometricFloat 25s infinite ease-in-out;
}

.shape-1 {
  width: 200px;
  height: 200px;
  top: 20%;
  left: 10%;
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  animation-delay: 0s;
}

.shape-2 {
  width: 150px;
  height: 150px;
  top: 60%;
  right: 15%;
  clip-path: polygon(25% 0%, 100% 0%, 75% 100%, 0% 100%);
  animation-delay: 8s;
}

.shape-3 {
  width: 100px;
  height: 100px;
  bottom: 20%;
  left: 60%;
  clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);
  animation-delay: 16s;
}

@keyframes geometricFloat {
  0%, 100% { transform: translateY(0px) rotate(0deg); }
  33% { transform: translateY(-20px) rotate(120deg); }
  66% { transform: translateY(20px) rotate(240deg); }
}

.app-header {
  position: relative;
  z-index: 10;
  text-align: center;
  padding: 50px 20px 30px;
  border-bottom: 2px solid rgba(255, 107, 53, 0.3);
}

h1 {
  color: #ffffff;
  font-size: clamp(2.5rem, 6vw, 4rem);
  font-weight: 900;
  text-shadow: 0 0 20px rgba(255, 107, 53, 0.5);
  letter-spacing: 0.05em;
  margin-bottom: 15px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
  text-transform: uppercase;
  font-family: 'Orbitron', 'Rajdhani', monospace;
}

.icon {
  font-size: 1.2em;
  display: block;
  color: #ff6b35;
  text-shadow: 0 0 30px #ff6b35;
  animation: electricPulse 2s infinite;
}

@keyframes electricPulse {
  0%, 100% { 
    transform: scale(1);
    text-shadow: 0 0 30px #ff6b35;
  }
  50% { 
    transform: scale(1.1);
    text-shadow: 0 0 40px #ff6b35, 0 0 60px #f7931e;
  }
}

.subtitle {
  font-size: 0.35em;
  font-weight: 700;
  opacity: 0.8;
  text-transform: uppercase;
  letter-spacing: 0.2em;
  margin-top: 10px;
  color: #f7931e;
  font-family: 'Rajdhani', sans-serif;
  border: 1px solid rgba(247, 147, 30, 0.3);
  padding: 8px 16px;
  background: rgba(247, 147, 30, 0.1);
}

.app-main {
  flex: 1;
  position: relative;
  z-index: 10;
  padding: 0 20px 50px;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}

/* Responsive */
@media (max-width: 768px) {
  .app-header {
    padding: 30px 15px 20px;
  }
  
  .app-main {
    padding: 0 15px 30px;
  }
  
  .shape {
    display: none;
  }
  
  .grid-line {
    opacity: 0.05;
  }
}
</style>
