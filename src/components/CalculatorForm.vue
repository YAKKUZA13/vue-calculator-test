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
  padding: 30px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #ffffff;
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: auto auto 1fr;
  gap: 30px;
  min-height: calc(100vh - 200px);
}

.input-section {
  grid-column: 1 / -1;
  display: flex;
  gap: 20px;
  padding: 30px;
  background-color: #f8f9fa;
  border-radius: 12px;
  border: 2px solid #e0e0e0;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}

.input-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 180px;
  max-width: 220px;
  flex: 1;
}

.input-field {
  padding: 15px 20px;
  border: 2px solid #333333;
  border-radius: 8px;
  font-size: 18px;
  font-weight: 500;
  min-width: 180px;
  max-width: 220px;
  flex: 1;
  background-color: #ffffff;
  color: #000000;
  transition: all 0.3s ease;
  text-align: center;
}

.input-field:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 4px rgba(0, 123, 255, 0.25);
  transform: translateY(-2px);
}

.input-field::placeholder {
  color: #666666;
  font-weight: 400;
}

.input-label {
  margin-top: 12px;
  font-size: 16px;
  font-weight: 600;
  color: #000000;
  text-align: center;
  padding: 8px;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 6px;
  min-height: 20px;
  width: 100%;
  border: 1px solid #ddd;
}

.submit-button {
  padding: 15px 30px;
  background-color: #007bff;
  color: #ffffff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 18px;
  font-weight: 600;
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 1px;
  min-width: 200px;
  white-space: nowrap;
}

.submit-button:hover:not(:disabled) {
  background-color: #0056b3;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 123, 255, 0.4);
}

.submit-button:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.labels-section {
  grid-column: 1;
  padding: 30px;
  background-color: #343a40;
  border-radius: 12px;
  border: 2px solid #495057;
  height: fit-content;
}

.localStorage-display {
  background-color: #495057 !important;
  border-radius: 8px;
  padding: 20px !important;
  margin: 0;
}

.localStorage-display pre {
  color: #ffffff;
  font-family: 'Courier New', monospace;
  font-size: 14px;
  white-space: pre-wrap;
  word-break: break-all;
  margin: 0;
  line-height: 1.5;
}

.events-section {
  grid-column: 2;
  border: 2px solid #333333;
  border-radius: 12px;
  padding: 30px;
  background-color: #ffffff;
  display: flex;
  flex-direction: column;
  height: fit-content;
  max-height: 70vh;
}

.events-section h3 {
  margin-top: 0;
  margin-bottom: 25px;
  color: #000000;
  font-size: 22px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1px;
  border-bottom: 3px solid #007bff;
  padding-bottom: 15px;
}

.events-list {
  flex: 1;
  overflow-y: auto;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  background-color: #f8f9fa;
  max-height: 500px;
}

.event-item {
  border-bottom: 2px solid #dee2e6;
  padding: 20px;
  margin: 0;
  background-color: #ffffff;
  margin-bottom: 2px;
  transition: background-color 0.2s ease;
}

.event-item:last-child {
  border-bottom: none;
  margin-bottom: 0;
}

.event-item:hover {
  background-color: #e3f2fd;
}

.event-timestamp {
  font-size: 12px;
  color: #000000;
  margin-bottom: 8px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.event-type {
  font-weight: 700;
  margin-bottom: 12px;
  color: #007bff;
  font-size: 16px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.event-data {
  font-size: 14px;
  word-break: break-all;
  background-color: #e9ecef;
  color: #000000;
  padding: 15px 20px;
  border-radius: 8px;
  font-weight: 500;
  line-height: 1.5;
  border: 2px solid #ced4da;
}

/* Адаптивность для мобильных устройств */
@media (max-width: 1024px) {
  .calculator-container {
    grid-template-columns: 1fr;
    padding: 20px;
    gap: 20px;
  }
  
  .labels-section,
  .events-section {
    grid-column: 1;
  }
  
  .input-section {
    flex-direction: column;
    align-items: stretch;
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
    padding: 15px;
    gap: 15px;
    margin: 10px;
  }
  
  .input-section {
    padding: 20px;
  }
  
  .labels-section,
  .events-section {
    padding: 20px;
  }
  
  .events-section {
    max-height: 50vh;
  }
  
  .input-field {
    font-size: 16px;
    padding: 12px 15px;
  }
  
  .submit-button {
    font-size: 16px;
    padding: 12px 20px;
  }
}
</style> 