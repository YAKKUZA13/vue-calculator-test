<script setup lang="ts">
import CalculatorForm from './components/CalculatorForm.vue'
</script>

<template>
  <div id="app">
    <div class="crt-overlay"></div>
    <div class="scanlines"></div>
    <div class="background-grid">
      <div class="grid-line" v-for="i in 25" :key="i"></div>
    </div>
    <main class="app-main">
      <CalculatorForm />
    </main>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=VT323&family=Courier+Prime&display=swap');

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  font-family: 'VT323', 'Courier Prime', monospace;
  background: #000000;
  overflow-x: hidden;
  cursor: none;
}

#app {
  min-height: 100vh;
  width: 100%;
  position: relative;
  background: #001100;
  display: flex;
  flex-direction: column;
  padding: 0;
  filter: contrast(1.2) brightness(1.1);
}

.crt-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: 
    radial-gradient(ellipse at center, transparent 0%, rgba(0, 40, 0, 0.3) 100%),
    linear-gradient(0deg, transparent 50%, rgba(0, 255, 0, 0.03) 50%);
  background-size: 100% 100%, 100% 4px;
  pointer-events: none;
  z-index: 1000;
  animation: crtFlicker 0.15s infinite linear alternate;
}

@keyframes crtFlicker {
  0% { opacity: 1; }
  98% { opacity: 1; }
  99% { opacity: 0.98; }
  100% { opacity: 1; }
}

.scanlines {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(0, 255, 0, 0.05) 2px,
    rgba(0, 255, 0, 0.05) 4px
  );
  pointer-events: none;
  z-index: 999;
  animation: scanlineMove 0.1s linear infinite;
}

@keyframes scanlineMove {
  0% { transform: translateY(0px); }
  100% { transform: translateY(4px); }
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
  background: #00ff00;
  height: 1px;
  width: 100%;
  animation: gridGlow 3s infinite ease-in-out;
}

.grid-line:nth-child(odd) {
  animation-delay: 0s;
}

.grid-line:nth-child(even) {
  animation-delay: 1.5s;
}

.grid-line:nth-child(1) { top: 4%; }
.grid-line:nth-child(2) { top: 8%; }
.grid-line:nth-child(3) { top: 12%; }
.grid-line:nth-child(4) { top: 16%; }
.grid-line:nth-child(5) { top: 20%; }
.grid-line:nth-child(6) { top: 24%; }
.grid-line:nth-child(7) { top: 28%; }
.grid-line:nth-child(8) { top: 32%; }
.grid-line:nth-child(9) { top: 36%; }
.grid-line:nth-child(10) { top: 40%; }
.grid-line:nth-child(11) { top: 44%; }
.grid-line:nth-child(12) { top: 48%; }
.grid-line:nth-child(13) { top: 52%; }
.grid-line:nth-child(14) { top: 56%; }
.grid-line:nth-child(15) { top: 60%; }
.grid-line:nth-child(16) { top: 64%; }
.grid-line:nth-child(17) { top: 68%; }
.grid-line:nth-child(18) { top: 72%; }
.grid-line:nth-child(19) { top: 76%; }
.grid-line:nth-child(20) { top: 80%; }
.grid-line:nth-child(21) { top: 84%; }
.grid-line:nth-child(22) { top: 88%; }
.grid-line:nth-child(23) { top: 92%; }
.grid-line:nth-child(24) { top: 96%; }
.grid-line:nth-child(25) { top: 100%; }

@keyframes gridGlow {
  0%, 100% { 
    opacity: 0.05;
    box-shadow: 0 0 2px #00ff00;
  }
  50% { 
    opacity: 0.15;
    box-shadow: 0 0 5px #00ff00;
  }
}

.app-main {
  flex: 1;
  position: relative;
  z-index: 10;
  padding: 40px 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Custom cursor */
body::after {
  content: '█';
  position: fixed;
  color: #00ff00;
  font-family: 'VT323', monospace;
  font-size: 20px;
  pointer-events: none;
  z-index: 9999;
  animation: blink 1s infinite;
  mix-blend-mode: screen;
}

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}

/* Responsive */
@media (max-width: 768px) {
  .app-main {
    padding: 20px 15px;
  }
  
  .grid-line {
    opacity: 0.03;
  }
  
  .crt-overlay {
    opacity: 0.7;
  }
}

/* Boot sequence animation */
#app {
  animation: bootSequence 2s ease-out;
}

@keyframes bootSequence {
  0% {
    background: #000000;
    filter: brightness(0);
  }
  30% {
    background: #000000;
    filter: brightness(0);
  }
  60% {
    background: #001100;
    filter: brightness(0.5) contrast(1);
  }
  100% {
    background: #001100;
    filter: contrast(1.2) brightness(1.1);
  }
}
</style>
