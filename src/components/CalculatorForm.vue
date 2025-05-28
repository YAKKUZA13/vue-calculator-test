<template>
  <div class="calculator-container">
    <div class="input-section">
      <div class="input-group">
        <input 
          v-model="price" 
          @input="onPriceChange"
          type="number" 
          placeholder="Цена"
          class="input-field"
        />
        <div class="input-label">Цена: {{ Number(price).toFixed(2) }}</div>
      </div>
      
      <div class="input-group">
        <input 
          v-model="qty" 
          @input="onQtyChange"
          type="number" 
          placeholder="Количество"
          class="input-field"
        />
        <div class="input-label">Количество: {{ Number(qty).toFixed(2) }}</div>
      </div>
      
      <div class="input-group">
        <input 
          v-model="amount" 
          @input="onAmountChange"
          type="number" 
          placeholder="Сумма"
          class="input-field"
        />
        <div class="input-label">Сумма: {{ Number(amount).toFixed(2) }}</div>
      </div>
      
      <button @click="submitData" :disabled="isLoading" class="submit-button">
        {{ isLoading ? 'Отправка...' : 'Отправить данные' }}
      </button>
    </div>

    <div class="labels-section">
      <div class="localStorage-display">LocalStorage: <pre>{{ localStorageData }}</pre></div>
    </div>

    <div class="events-section">
      <h3>События:</h3>
      <div class="events-list">
        <div v-for="event in events" :key="event.id" class="event-item">
          <div class="event-timestamp">{{ event.timestamp }}</div>
          <div class="event-type">{{ event.type }}</div>
          <div class="event-data">{{ event.data }}</div>
        </div>
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
  max-width: 1400px;
  margin: 0 auto;
  padding: 40px;
  font-family: 'Rajdhani', 'Orbitron', 'Roboto Condensed', sans-serif;
  background: rgba(20, 20, 20, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 4px;
  border: 2px solid rgba(255, 107, 53, 0.3);
  box-shadow: 0 0 40px rgba(255, 107, 53, 0.2),
              inset 0 1px 0 rgba(255, 107, 53, 0.1);
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: auto auto 1fr;
  gap: 30px;
  min-height: calc(100vh - 300px);
  position: relative;
  overflow: hidden;
}

.calculator-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg, 
    rgba(255, 107, 53, 0.03) 0%,
    rgba(20, 20, 20, 0.8) 50%,
    rgba(255, 107, 53, 0.03) 100%);
  pointer-events: none;
  z-index: 1;
}

.calculator-container > * {
  position: relative;
  z-index: 2;
}

.input-section {
  grid-column: 1 / -1;
  display: flex;
  gap: 25px;
  padding: 40px;
  background: rgba(30, 30, 30, 0.9);
  backdrop-filter: blur(10px);
  border-radius: 2px;
  border: 1px solid rgba(255, 107, 53, 0.4);
  box-shadow: inset 0 2px 0 rgba(255, 107, 53, 0.2),
              0 4px 20px rgba(0, 0, 0, 0.5);
  flex-wrap: wrap;
  justify-content: center;
  align-items: flex-start;
  position: relative;
  overflow: hidden;
}

.input-section::after {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, 
    transparent, 
    #ff6b35, 
    #f7931e,
    transparent);
  animation: scanLine 4s infinite;
}

@keyframes scanLine {
  0% { left: -100%; }
  100% { left: 100%; }
}

.input-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 200px;
  max-width: 250px;
  flex: 1;
  margin-bottom: 20px;
}

.input-field {
  width: 100%;
  padding: 18px 24px;
  border: 2px solid rgba(255, 107, 53, 0.3);
  border-radius: 2px;
  font-size: 18px;
  font-weight: 700;
  background: rgba(15, 15, 15, 0.9);
  color: #ffffff;
  text-align: center;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.3),
              0 0 20px rgba(255, 107, 53, 0.1);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  font-family: 'Orbitron', monospace;
  text-transform: uppercase;
}

.input-field:focus {
  outline: none;
  transform: translateY(-2px);
  border-color: #ff6b35;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.3),
              0 0 30px rgba(255, 107, 53, 0.4),
              0 0 0 2px rgba(255, 107, 53, 0.2);
  background: rgba(20, 20, 20, 0.95);
}

.input-field::placeholder {
  color: rgba(255, 107, 53, 0.5);
  font-weight: 600;
}

.input-label {
  margin-top: 16px;
  font-size: 14px;
  font-weight: 700;
  color: #ff6b35;
  text-align: center;
  padding: 10px 16px;
  background: rgba(30, 30, 30, 0.8);
  backdrop-filter: blur(10px);
  border-radius: 2px;
  width: 100%;
  border: 1px solid rgba(255, 107, 53, 0.3);
  box-shadow: inset 0 1px 0 rgba(255, 107, 53, 0.1),
              0 2px 10px rgba(0, 0, 0, 0.3);
  text-shadow: 0 0 10px rgba(255, 107, 53, 0.3);
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  font-family: 'Rajdhani', sans-serif;
}

.input-label:hover {
  background: rgba(40, 40, 40, 0.9);
  transform: translateY(-1px);
  box-shadow: inset 0 1px 0 rgba(255, 107, 53, 0.2),
              0 4px 15px rgba(0, 0, 0, 0.4);
}

.submit-button {
  padding: 18px 40px;
  background: linear-gradient(135deg, #ff6b35 0%, #f7931e 100%);
  color: #000000;
  border: 2px solid #ff6b35;
  border-radius: 2px;
  cursor: pointer;
  font-size: 18px;
  font-weight: 900;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  min-width: 220px;
  white-space: nowrap;
  position: relative;
  overflow: hidden;
  box-shadow: 0 0 30px rgba(255, 107, 53, 0.4),
              inset 0 1px 0 rgba(255, 255, 255, 0.2);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  font-family: 'Orbitron', sans-serif;
}

.submit-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, 
    transparent, 
    rgba(255, 255, 255, 0.3), 
    transparent);
  transition: left 0.5s;
}

.submit-button:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 0 40px rgba(255, 107, 53, 0.6),
              inset 0 1px 0 rgba(255, 255, 255, 0.3);
  background: linear-gradient(135deg, #f7931e 0%, #ff6b35 100%);
}

.submit-button:hover:not(:disabled)::before {
  left: 100%;
}

.submit-button:active:not(:disabled) {
  transform: translateY(-1px);
}

.submit-button:disabled {
  background: linear-gradient(135deg, #666666 0%, #444444 100%);
  border-color: #666666;
  color: #999999;
  cursor: not-allowed;
  transform: none;
  box-shadow: 0 0 15px rgba(102, 102, 102, 0.2);
}

.labels-section {
  grid-column: 1;
  padding: 30px;
  background: rgba(20, 20, 20, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 2px;
  border: 1px solid rgba(255, 107, 53, 0.3);
  box-shadow: 0 0 30px rgba(255, 107, 53, 0.1);
  height: fit-content;
  position: relative;
  overflow: hidden;
}

.labels-section::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, 
    transparent,
    #ff6b35,
    #f7931e,
    transparent);
}

.localStorage-display {
  background: rgba(30, 30, 30, 0.9) !important;
  backdrop-filter: blur(10px);
  border-radius: 2px;
  padding: 24px !important;
  margin: 0;
  border: 1px solid rgba(255, 107, 53, 0.4);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.3),
              0 0 20px rgba(255, 107, 53, 0.1);
  position: relative;
  overflow: hidden;
}

.localStorage-display::before {
  content: '💾 STORAGE';
  position: absolute;
  top: 8px;
  right: 12px;
  font-size: 12px;
  color: #ff6b35;
  font-weight: 700;
  opacity: 0.7;
  font-family: 'Rajdhani', sans-serif;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

.localStorage-display pre {
  color: #ff6b35;
  font-family: 'Orbitron', 'JetBrains Mono', monospace;
  font-size: 14px;
  white-space: pre-wrap;
  word-break: break-all;
  margin: 0;
  line-height: 1.6;
  text-shadow: 0 0 10px rgba(255, 107, 53, 0.3);
  font-weight: 600;
}

.events-section {
  grid-column: 2;
  border: 1px solid rgba(255, 107, 53, 0.3);
  border-radius: 2px;
  padding: 30px;
  background: rgba(20, 20, 20, 0.95);
  backdrop-filter: blur(20px);
  display: flex;
  flex-direction: column;
  height: fit-content;
  max-height: 70vh;
  box-shadow: 0 0 30px rgba(255, 107, 53, 0.1);
  position: relative;
  overflow: hidden;
}

.events-section::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, 
    transparent,
    #f7931e,
    #ff6b35,
    transparent);
}

.events-section h3 {
  margin-top: 0;
  margin-bottom: 25px;
  color: #ff6b35;
  font-size: 22px;
  font-weight: 900;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  text-align: center;
  padding-bottom: 15px;
  border-bottom: 2px solid rgba(255, 107, 53, 0.3);
  text-shadow: 0 0 20px rgba(255, 107, 53, 0.5);
  position: relative;
  font-family: 'Orbitron', sans-serif;
}

.events-section h3::before {
  content: '⚡ EVENTS LOG';
  font-size: 14px;
  color: #f7931e;
}

.events-list {
  flex: 1;
  overflow-y: auto;
  border-radius: 2px;
  background: rgba(15, 15, 15, 0.8);
  backdrop-filter: blur(10px);
  max-height: 500px;
  padding: 5px;
  border: 1px solid rgba(255, 107, 53, 0.2);
}

.events-list::-webkit-scrollbar {
  width: 8px;
}

.events-list::-webkit-scrollbar-track {
  background: rgba(20, 20, 20, 0.5);
  border-radius: 2px;
}

.events-list::-webkit-scrollbar-thumb {
  background: rgba(255, 107, 53, 0.5);
  border-radius: 2px;
}

.events-list::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 107, 53, 0.7);
}

.event-item {
  border-bottom: 1px solid rgba(255, 107, 53, 0.2);
  padding: 20px;
  margin: 5px;
  background: rgba(30, 30, 30, 0.8);
  backdrop-filter: blur(10px);
  border-radius: 2px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid rgba(255, 107, 53, 0.1);
  position: relative;
  overflow: hidden;
}

.event-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, 
    transparent, 
    rgba(255, 107, 53, 0.1), 
    transparent);
  transition: left 0.5s;
}

.event-item:hover {
  background: rgba(40, 40, 40, 0.9);
  transform: translateX(5px);
  border-color: rgba(255, 107, 53, 0.4);
  box-shadow: 0 0 20px rgba(255, 107, 53, 0.1);
}

.event-item:hover::before {
  left: 100%;
}

.event-item:last-child {
  border-bottom: none;
  margin-bottom: 0;
}

.event-timestamp {
  font-size: 11px;
  color: rgba(247, 147, 30, 0.8);
  margin-bottom: 8px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-family: 'Orbitron', monospace;
}

.event-type {
  font-weight: 900;
  margin-bottom: 12px;
  color: #ff6b35;
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  text-shadow: 0 0 10px rgba(255, 107, 53, 0.3);
  font-family: 'Rajdhani', sans-serif;
}

.event-data {
  font-size: 13px;
  word-break: break-all;
  background: rgba(15, 15, 15, 0.8);
  backdrop-filter: blur(5px);
  color: #ffffff;
  padding: 15px 20px;
  border-radius: 2px;
  font-weight: 500;
  line-height: 1.5;
  border: 1px solid rgba(255, 107, 53, 0.2);
  font-family: 'Orbitron', monospace;
  text-shadow: 0 0 5px rgba(255, 255, 255, 0.1);
}

/* Адаптивность для мобильных устройств */
@media (max-width: 1024px) {
  .calculator-container {
    grid-template-columns: 1fr;
    padding: 30px;
    gap: 25px;
  }
  
  .labels-section,
  .events-section {
    grid-column: 1;
  }
  
  .input-section {
    flex-direction: column;
    align-items: stretch;
    padding: 30px;
  }
  
  .input-group {
    min-width: auto;
    max-width: none;
    width: 100%;
  }
  
  .input-field,
  .submit-button {
    min-width: auto;
    width: 100%;
  }
}

@media (max-width: 768px) {
  .calculator-container {
    padding: 20px;
    gap: 20px;
    margin: 10px;
    border-radius: 2px;
  }
  
  .input-section {
    padding: 25px;
  }
  
  .labels-section,
  .events-section {
    padding: 25px;
  }
  
  .events-section {
    max-height: 50vh;
  }
  
  .input-field {
    font-size: 16px;
    padding: 15px 20px;
  }
  
  .submit-button {
    font-size: 16px;
    padding: 15px 25px;
  }
  
  .input-group {
    margin-bottom: 15px;
  }
}

/* Военные анимации */
@keyframes tacticalPulse {
  0%, 100% { 
    box-shadow: 0 0 30px rgba(255, 107, 53, 0.2);
  }
  50% { 
    box-shadow: 0 0 40px rgba(255, 107, 53, 0.4);
  }
}

.submit-button:disabled {
  animation: tacticalPulse 2s infinite;
}

/* Анимация появления */
.calculator-container {
  animation: militaryDeploy 1s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes militaryDeploy {
  from {
    opacity: 0;
    transform: translateY(30px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.input-group {
  animation: squadDeploy 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

.input-group:nth-child(1) { animation-delay: 0.1s; }
.input-group:nth-child(2) { animation-delay: 0.2s; }
.input-group:nth-child(3) { animation-delay: 0.3s; }
.submit-button { animation-delay: 0.4s; }

@keyframes squadDeploy {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}
</style> 