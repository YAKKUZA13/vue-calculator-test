<template>
  <div class="calculator-container">
    <div class="terminal-header">
      <div class="terminal-title">]APPLE ][ CALCULATOR SYSTEM</div>
      <div class="terminal-cursor">█</div>
    </div>
    
    <div class="input-section">
      <div class="terminal-prompt">]INPUT VALUES:</div>
      
      <div class="input-group">
        <div class="input-label-top">]PRICE:</div>
        <input 
          v-model="price" 
          @input="onPriceChange"
          type="number" 
          placeholder="0.00"
          class="input-field"
        />
        <div class="input-display">PRICE: {{ Number(price).toFixed(2) }}</div>
      </div>
      
      <div class="input-group">
        <div class="input-label-top">]QTY:</div>
        <input 
          v-model="qty" 
          @input="onQtyChange"
          type="number" 
          placeholder="0.00"
          class="input-field"
        />
        <div class="input-display">QTY: {{ Number(qty).toFixed(2) }}</div>
      </div>
      
      <div class="input-group">
        <div class="input-label-top">]AMOUNT:</div>
        <input 
          v-model="amount" 
          @input="onAmountChange"
          type="number" 
          placeholder="0.00"
          class="input-field"
        />
        <div class="input-display">AMOUNT: {{ Number(amount).toFixed(2) }}</div>
      </div>
      
      <div class="submit-section">
        <div class="terminal-prompt">]EXECUTE:</div>
        <button @click="submitData" :disabled="isLoading" class="submit-button">
          {{ isLoading ? '>PROCESSING...' : '>SUBMIT DATA' }}
        </button>
      </div>
    </div>

    <div class="data-section">
      <div class="storage-panel">
        <div class="panel-header">┌─ STORAGE ─────────────────────┐</div>
        <div class="panel-content">
          <div class="storage-display">{{ localStorageData }}</div>
        </div>
        <div class="panel-footer">└───────────────────────────────┘</div>
      </div>

      <div class="events-panel">
        <div class="panel-header">┌─ EVENT LOG ───────────────────┐</div>
        <div class="panel-content">
          <div class="events-list">
            <div v-for="event in events" :key="event.id" class="event-item">
              <div class="event-line">
                >{{ event.timestamp }} [{{ event.type.toUpperCase() }}]
              </div>
              <div class="event-data">{{ event.data }}</div>
            </div>
          </div>
        </div>
        <div class="panel-footer">└───────────────────────────────┘</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

// Типы для событий
interface Event {
  id: number
  timestamp: string
  type: string
  data: string
}

interface LocalStorageData {
  counter: number
  price: number
  qty: number
  amount: number
}

// Реактивные данные
const price = ref<number>(0)
const qty = ref<number>(0)
const amount = ref<number>(0)
const counter = ref<number>(0)
const isLoading = ref<boolean>(false)
const events = ref<Event[]>([])
const lastChangedField = ref<string>('')
const fieldChangeOrder = ref<string[]>([]) // Порядок изменения полей

// Debounce таймеры
let priceDebounceTimer: number | null = null
let qtyDebounceTimer: number | null = null
let amountDebounceTimer: number | null = null

// Вычисляемое свойство для отображения localStorage
const localStorageData = computed(() => {
  // Используем forceUpdate для принудительного обновления
  forceUpdate.value
  
  const data = localStorage.getItem('calculatorData')
  if (data) {
    try {
      const parsed = JSON.parse(data)
      return JSON.stringify(parsed, null, 2)
    } catch (e) {
      return data
    }
  }
  return '{}'
})

// Реактивное свойство для принудительного обновления localStorage
const forceUpdate = ref<number>(0)

// Функция для принудительного обновления localStorage отображения
const updateLocalStorageDisplay = () => {
  forceUpdate.value++
}

// Функции для обработки изменений полей с debounce
const onPriceChange = () => {
  if (priceDebounceTimer) {
    window.clearTimeout(priceDebounceTimer)
  }
  priceDebounceTimer = window.setTimeout(() => {
    updateFieldChangeOrder('price')
    calculateValues()
    addEvent('input_change', `Изменена цена: ${price.value}`)
  }, 300)
}

const onQtyChange = () => {
  if (qtyDebounceTimer) {
    window.clearTimeout(qtyDebounceTimer)
  }
  qtyDebounceTimer = window.setTimeout(() => {
    updateFieldChangeOrder('qty')
    calculateValues()
    addEvent('input_change', `Изменено количество: ${qty.value}`)
  }, 300)
}

const onAmountChange = () => {
  if (amountDebounceTimer) {
    window.clearTimeout(amountDebounceTimer)
  }
  amountDebounceTimer = window.setTimeout(() => {
    updateFieldChangeOrder('amount')
    calculateValues()
    addEvent('input_change', `Изменена сумма: ${amount.value}`)
  }, 300)
}

// Функция для обновления порядка изменения полей
const updateFieldChangeOrder = (field: string) => {
  // Удаляем поле из массива если оно уже есть
  const index = fieldChangeOrder.value.indexOf(field)
  if (index > -1) {
    fieldChangeOrder.value.splice(index, 1)
  }
  // Добавляем в конец как самое последнее измененное
  fieldChangeOrder.value.push(field)
  lastChangedField.value = field
}

// Функция для пересчета значений
const calculateValues = () => {
  const changedField = lastChangedField.value
  
  if (changedField === 'price' || changedField === 'qty') {
    // Если изменились цена или количество - пересчитываем сумму
    const newAmount = Number((Number(price.value) * Number(qty.value)).toFixed(2))
    amount.value = newAmount
  } else if (changedField === 'amount') {
    // Если изменилась сумма - пересчитываем поле, которое менялось раньше всех
    if (fieldChangeOrder.value.length >= 2) {
      // Находим поле, которое менялось раньше (не считая amount)
      const earliestField = fieldChangeOrder.value
        .filter(f => f !== 'amount')
        .shift() // Берем самое раннее
      
      if (earliestField === 'price' && Number(qty.value) !== 0) {
        price.value = Number((Number(amount.value) / Number(qty.value)).toFixed(2))
      } else if (earliestField === 'qty' && Number(price.value) !== 0) {
        qty.value = Number((Number(amount.value) / Number(price.value)).toFixed(2))
      }
    } else {
      // Если история изменений короткая, пересчитываем qty по умолчанию
      if (Number(price.value) !== 0) {
        qty.value = Number((Number(amount.value) / Number(price.value)).toFixed(2))
      }
    }
  }
}

// Функция для добавления события
const addEvent = (type: string, data: string) => {
  const event: Event = {
    id: Date.now(),
    timestamp: new Date().toLocaleTimeString(),
    type,
    data
  }
  events.value.unshift(event) // Добавляем в начало массива (свежие события сверху)
}

// Функция отправки данных
const submitData = async () => {
  isLoading.value = true
  
  const dataToSubmit = {
    counter: counter.value + 1,
    price: Number(price.value),
    qty: Number(qty.value),
    amount: Number(amount.value)
  }

  const currentLocalStorage = localStorage.getItem('calculatorData')
  
  addEvent('button_click', `Отправлено: ${JSON.stringify(dataToSubmit)}. LocalStorage на момент нажатия: ${currentLocalStorage || '{}'}`)

  // Имитация запроса к backend с задержкой 1 секунда
  try {
    await new Promise(resolve => window.setTimeout(resolve, 1000))
    
    // Проверяем четность суммы для определения успеха
    const isSuccess = Number(amount.value) % 2 === 0
    const response = { success: isSuccess }

    if (isSuccess) {
      // Сохраняем данные в localStorage только при успехе
      counter.value += 1
      const dataToSave: LocalStorageData = {
        counter: counter.value,
        price: Number(price.value),
        qty: Number(qty.value),
        amount: Number(amount.value)
      }
      localStorage.setItem('calculatorData', JSON.stringify(dataToSave))
    }

    // Принудительно обновляем отображение localStorage
    updateLocalStorageDisplay()
    
    const currentLocalStorageAfter = localStorage.getItem('calculatorData')
    addEvent('response_received', `Ответ: ${JSON.stringify(response)}. LocalStorage после ответа: ${currentLocalStorageAfter || '{}'}`)
    
  } catch (error) {
    addEvent('error', `Ошибка при отправке: ${error}`)
  } finally {
    isLoading.value = false
  }
}

// Инициализация при монтировании компонента
onMounted(() => {
  const savedData = localStorage.getItem('calculatorData')
  if (savedData) {
    try {
      const parsed: LocalStorageData = JSON.parse(savedData)
      counter.value = parsed.counter || 0
      price.value = Number(parsed.price) || 0
      qty.value = Number(parsed.qty) || 0
      amount.value = Number(parsed.amount) || 0
    } catch (e) {
      console.error('Ошибка при парсинге localStorage:', e)
    }
  }
  
  // Принудительно обновляем отображение localStorage
  updateLocalStorageDisplay()
  
  addEvent('component_mounted', 'Компонент загружен')
})
</script>

<style scoped>
.calculator-container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'VT323', 'Courier Prime', monospace;
  background: #001100;
  border: 2px solid #00ff00;
  border-radius: 0;
  color: #00ff00;
  font-size: 18px;
  line-height: 1.2;
  text-transform: uppercase;
  position: relative;
  box-shadow: 
    inset 0 0 50px rgba(0, 255, 0, 0.1),
    0 0 30px rgba(0, 255, 0, 0.3);
  animation: terminalGlow 3s ease infinite alternate;
}

@keyframes terminalGlow {
  0% { 
    box-shadow: 
      inset 0 0 50px rgba(0, 255, 0, 0.1),
      0 0 30px rgba(0, 255, 0, 0.3);
  }
  100% { 
    box-shadow: 
      inset 0 0 70px rgba(0, 255, 0, 0.15),
      0 0 40px rgba(0, 255, 0, 0.4);
  }
}

.terminal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #00ff00;
  margin-bottom: 20px;
}

.terminal-title {
  font-size: 20px;
  font-weight: bold;
  letter-spacing: 2px;
  text-shadow: 0 0 10px #00ff00;
}

.terminal-cursor {
  animation: blink 1s infinite;
  font-size: 20px;
}

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}

.terminal-prompt {
  margin: 15px 0 10px 0;
  font-size: 16px;
  color: #00ff00;
  text-shadow: 0 0 5px #00ff00;
}

.input-section {
  margin-bottom: 30px;
  padding: 20px;
  border: 1px solid #00ff00;
  background: rgba(0, 255, 0, 0.02);
}

.input-group {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.input-label-top {
  font-size: 14px;
  color: #00ff00;
  text-shadow: 0 0 5px #00ff00;
  margin-bottom: 5px;
}

.input-field {
  width: 100%;
  padding: 8px 12px;
  background: #000000;
  border: 1px solid #00ff00;
  color: #00ff00;
  font-family: 'VT323', monospace;
  font-size: 18px;
  text-transform: uppercase;
  outline: none;
  box-shadow: inset 0 0 10px rgba(0, 255, 0, 0.1);
}

.input-field:focus {
  box-shadow: 
    inset 0 0 20px rgba(0, 255, 0, 0.2),
    0 0 10px rgba(0, 255, 0, 0.5);
  text-shadow: 0 0 5px #00ff00;
}

.input-field::placeholder {
  color: rgba(0, 255, 0, 0.5);
}

.input-display {
  font-size: 14px;
  color: #00ff00;
  background: rgba(0, 255, 0, 0.05);
  padding: 5px 10px;
  border: 1px solid rgba(0, 255, 0, 0.3);
  margin-top: 5px;
  font-family: 'VT323', monospace;
}

.submit-section {
  margin-top: 20px;
  padding-top: 15px;
  border-top: 1px solid rgba(0, 255, 0, 0.3);
}

.submit-button {
  background: #000000;
  border: 2px solid #00ff00;
  color: #00ff00;
  padding: 10px 20px;
  font-family: 'VT323', monospace;
  font-size: 16px;
  text-transform: uppercase;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: inset 0 0 10px rgba(0, 255, 0, 0.1);
}

.submit-button:hover:not(:disabled) {
  background: rgba(0, 255, 0, 0.1);
  box-shadow: 
    inset 0 0 20px rgba(0, 255, 0, 0.2),
    0 0 15px rgba(0, 255, 0, 0.5);
  text-shadow: 0 0 8px #00ff00;
}

.submit-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  animation: processingBlink 0.5s infinite;
}

@keyframes processingBlink {
  0%, 50% { background: #000000; }
  51%, 100% { background: rgba(0, 255, 0, 0.1); }
}

.data-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-top: 30px;
}

.storage-panel,
.events-panel {
  background: rgba(0, 255, 0, 0.02);
  border: 1px solid #00ff00;
  font-family: 'VT323', monospace;
}

.panel-header,
.panel-footer {
  color: #00ff00;
  font-size: 14px;
  text-align: center;
  padding: 5px;
  background: rgba(0, 255, 0, 0.05);
  text-shadow: 0 0 5px #00ff00;
}

.panel-content {
  padding: 15px;
  min-height: 200px;
  max-height: 400px;
  overflow-y: auto;
}

.storage-display {
  font-family: 'VT323', monospace;
  font-size: 14px;
  color: #00ff00;
  white-space: pre-wrap;
  word-break: break-all;
  line-height: 1.4;
  background: rgba(0, 255, 0, 0.02);
  padding: 10px;
  border: 1px solid rgba(0, 255, 0, 0.2);
}

.events-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.event-item {
  border-bottom: 1px solid rgba(0, 255, 0, 0.2);
  padding-bottom: 10px;
  margin-bottom: 10px;
}

.event-item:last-child {
  border-bottom: none;
  margin-bottom: 0;
}

.event-line {
  font-size: 12px;
  color: #00ff00;
  margin-bottom: 5px;
  text-shadow: 0 0 3px #00ff00;
}

.event-data {
  font-size: 12px;
  color: rgba(0, 255, 0, 0.8);
  padding-left: 10px;
  word-break: break-all;
  line-height: 1.3;
  background: rgba(0, 255, 0, 0.02);
  border-left: 2px solid rgba(0, 255, 0, 0.3);
  padding: 5px 10px;
}

/* Scrollbar styles */
.panel-content::-webkit-scrollbar {
  width: 6px;
}

.panel-content::-webkit-scrollbar-track {
  background: rgba(0, 255, 0, 0.1);
}

.panel-content::-webkit-scrollbar-thumb {
  background: rgba(0, 255, 0, 0.5);
}

.panel-content::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 255, 0, 0.7);
}

/* Responsive design */
@media (max-width: 1024px) {
  .data-section {
    grid-template-columns: 1fr;
    gap: 15px;
  }
  
  .calculator-container {
    padding: 15px;
    font-size: 16px;
  }
}

@media (max-width: 768px) {
  .calculator-container {
    padding: 10px;
    font-size: 14px;
    margin: 10px;
  }
  
  .terminal-title {
    font-size: 16px;
  }
  
  .input-field {
    font-size: 16px;
    padding: 6px 10px;
  }
  
  .submit-button {
    font-size: 14px;
    padding: 8px 16px;
  }
  
  .panel-content {
    max-height: 250px;
  }
}

/* Terminal startup animation */
.calculator-container {
  animation: terminalStartup 2s ease-out;
}

@keyframes terminalStartup {
  0% {
    opacity: 0;
    transform: scale(0.8);
    filter: brightness(0);
  }
  50% {
    opacity: 0.5;
    transform: scale(0.9);
    filter: brightness(0.5);
  }
  100% {
    opacity: 1;
    transform: scale(1);
    filter: brightness(1);
  }
}

.input-group {
  animation: typeIn 0.5s ease-out;
}

.input-group:nth-child(2) { animation-delay: 0.1s; }
.input-group:nth-child(3) { animation-delay: 0.2s; }
.input-group:nth-child(4) { animation-delay: 0.3s; }

@keyframes typeIn {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* CRT scan effect */
.calculator-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(0, 255, 0, 0.03) 2px,
    rgba(0, 255, 0, 0.03) 4px
  );
  pointer-events: none;
  animation: scanMove 0.1s linear infinite;
}

@keyframes scanMove {
  0% { transform: translateY(0px); }
  100% { transform: translateY(4px); }
}

/* Retro glow effect */
.terminal-title,
.terminal-prompt,
.input-label-top {
  position: relative;
}

.terminal-title::after,
.terminal-prompt::after,
.input-label-top::after {
  content: attr(data-text);
  position: absolute;
  left: 0;
  top: 0;
  color: rgba(0, 255, 0, 0.3);
  filter: blur(1px);
  z-index: -1;
}
</style> 