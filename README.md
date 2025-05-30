# Vue.js Calculator App

Тестовое задание на Vue.js - калькулятор с отслеживанием событий и localStorage.

## Описание

Приложение реализует следующий функционал:

1. **Три поля ввода**: цена, количество, сумма
2. **Автоматический пересчет**: при изменении любого поля пересчитываются остальные согласно логике
3. **Debounce**: изменения в полях обрабатываются с задержкой 300ms
4. **Отправка данных**: кнопка имитирует отправку на backend с задержкой 1 секунда
5. **LocalStorage**: сохранение данных с монотонно возрастающим счетчиком
6. **Отслеживание событий**: все события отображаются в реальном времени

## Особенности логики

- **Умный пересчет**: поле, которое менялось раньше всех, пересчитывается при изменении других полей
- **Условная отправка**: данные сохраняются только если сумма четная
- **История событий**: все изменения и действия пользователя логируются

## Установка и запуск

```bash
# Установка зависимостей
npm install

# Запуск в режиме разработки
npm run dev

# Сборка для продакшена
npm run build

# Предварительный просмотр сборки
npm run preview
```

## Технологии

- Vue 3 с Composition API
- TypeScript
- Vite
- CSS3

## Структура проекта

```
src/
├── components/
│   └── CalculatorForm.vue  # Основной компонент калькулятора
├── App.vue                 # Корневой компонент
└── main.ts                 # Точка входа

```

## Особенности реализации

### Пересчет полей

Когда пользователь изменяет поле, система отслеживает порядок изменений и при изменении суммы пересчитывает то поле, которое менялось раньше всего.

### Отслеживание событий

Все события отображаются в обратном порядке (свежие сверху):
- Изменение полей ввода
- Нажатие кнопки отправки
- Получение ответа от "backend"

### LocalStorage

Структура данных в localStorage:
```json
{
  "counter": 1,
  "price": 100,
  "qty": 2,
  "amount": 200
}
```

## GitHub Pages

Проект можно развернуть на GitHub Pages следующим образом:

1. Создать репозиторий на GitHub
2. Загрузить код
3. Настроить GitHub Actions для автоматической сборки и развертывания
4. Включить GitHub Pages в настройках репозитория

## Автор

Тестовое задание для демонстрации навыков работы с Vue.js 3 и TypeScript.
