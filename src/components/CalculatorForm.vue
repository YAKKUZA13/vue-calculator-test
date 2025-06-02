<template>
  <div class="calculator-container">
    <!-- Строка с полями ввода и кнопкой -->
    <div class="input-row">
      <div class="input-group">
        <label>Цена:</label>
        <input 
          v-model="price" 
          @input="onPriceChange"
          type="number" 
          step="0.01"
          placeholder="0.00"
        />
      </div>
      
      <div class="input-group">
        <label>Количество:</label>
        <input 
          v-model="qty" 
          @input="onQtyChange"
          type="number" 
          step="0.01"
          placeholder="0.00"
        />
      </div>
      
      <div class="input-group">
        <label>Сумма:</label>
        <input 
          v-model="amount" 
          @input="onAmountChange"
          type="number" 
          step="0.01"
          placeholder="0.00"
        />
      </div>
      
      <button @click="submitData" :disabled="isLoading" class="submit-button">
        {{ isLoading ? 'Обработка...' : 'Отправить' }}
      </button>
    </div>

    <!-- Строка с лейблами -->
    <div class="labels-row">
      <div class="label-item">Цена: {{ Number(price).toFixed(2) }}</div>
      <div class="label-item">Количество: {{ Number(qty).toFixed(2) }}</div>
      <div class="label-item">Сумма: {{ Number(amount).toFixed(2) }}</div>
      <div class="label-item">localStorage: {{ localStorageData }}</div>
    </div>

    <!-- Область событий -->
    <div class="events-section">
      <h3>События:</h3>
      <div class="events-list">
        <div v-for="event in events" :key="event.id" class="event-item">
          <div class="event-timestamp">{{ event.timestamp }} [{{ event.type.toUpperCase() }}]</div>
          <div class="event-data">{{ event.data }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

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
  max-width: 1000px;
  padding: 20px;
  border: 1px solid #ccc;
  background: white;
  font-family: Arial, sans-serif;
}

.input-row {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  align-items: flex-end;
}

.input-group {
  display: flex;
  flex-direction: column;
  min-width: 120px;
}

.input-group label {
  margin-bottom: 5px;
  font-weight: bold;
}

.input-group input {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

.submit-button {
  padding: 8px 16px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  height: fit-content;
}

.submit-button:hover:not(:disabled) {
  background: #0056b3;
}

.submit-button:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

.labels-row {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
  padding: 10px;
  background: #f8f9fa;
  border: 1px solid #dee2e6;
  border-radius: 4px;
}

.label-item {
  font-size: 14px;
  color: #000000;
}

.events-section {
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
}

.events-section h3 {
  margin: 0;
  padding: 10px;
  background: #f8f9fa;
  border-bottom: 1px solid #dee2e6;
  font-size: 16px;
}

.events-list {
  max-height: 400px;
  overflow-y: auto;
  padding: 10px;
}

.event-item {
  margin-bottom: 15px;
  padding: 10px;
  border: 1px solid #e9ecef;
  border-radius: 4px;
  background: #f8f9fa;
}

.event-item:last-child {
  margin-bottom: 0;
}

.event-timestamp {
  font-size: 12px;
  color: #666;
  margin-bottom: 5px;
  font-weight: bold;
}

.event-data {
  font-size: 12px;
  color: #333;
  word-break: break-all;
  line-height: 1.4;
}

@media (max-width: 768px) {
  .input-row {
    flex-direction: column;
    gap: 10px;
  }
  
  .labels-row {
    flex-direction: column;
    gap: 5px;
  }
  
  .calculator-container {
    padding: 15px;
  }
}
</style> 