<template>
  <div class="form-page">
    <h1>Регистрация</h1>

    <!-- novalidate отключает встроенные проверки браузера: ошибки показываем сами -->
    <form @submit="handleSubmit" class="registration-form" novalidate>
      <!-- Имя -->
      <div class="form-group">
        <!-- for у label совпадает с id у input: клик по подписи ставит курсор в поле -->
        <label for="name">Имя</label>
        <!-- @blur - пользователь ушёл с поля, @input - ввёл или стёр символ -->
        <input
          id="name"
          v-model="form.name"
          type="text"
          class="form-input"
          :class="{ 'input-error': errors.name }"
          @blur="onBlur('name')"
          @input="onInput('name')"
        />
        <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
      </div>

      <!-- Email -->
      <div class="form-group">
        <label for="email">Email</label>
        <input
          id="email"
          v-model="form.email"
          type="email"
          class="form-input"
          :class="{ 'input-error': errors.email }"
          @blur="onBlur('email')"
          @input="onInput('email')"
        />
        <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
      </div>

      <!-- Пароль -->
      <div class="form-group">
        <label for="password">Пароль</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.password }"
          @blur="onBlur('password')"
          @input="onInput('password')"
        />
        <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
        <span v-else class="hint">
          Минимум 8 символов: строчные и заглавные буквы, цифры и спецсимволы
        </span>
      </div>

      <!-- Подтверждение пароля -->
      <div class="form-group">
        <label for="confirmPassword">Подтверждение пароля</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.confirmPassword }"
          @blur="onBlur('confirmPassword')"
          @input="onInput('confirmPassword')"
        />
        <span v-if="errors.confirmPassword" class="error-message">
          {{ errors.confirmPassword }}
        </span>
      </div>

      <!-- Общее сообщение об ошибке -->
      <div v-if="generalError" class="general-error">{{ generalError }}</div>

      <!-- Кнопка отправки. disabled - пока идёт отправка, второй раз нажать нельзя -->
      <button type="submit" class="btn-submit" :disabled="isLoading">
        {{ isLoading ? 'Отправка...' : 'Зарегистрироваться' }}
      </button>
    </form>

    <!-- Сообщение об успехе -->
    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import axios from 'axios'

// Тестовый адрес для отправки формы.
// Чтобы проверить ошибку сервера, можно поменять адрес на несуществующий,
// например 'https://jsonplaceholder.typicode.com/wrong-url' - сервер ответит кодом 404
const API_URL = 'https://jsonplaceholder.typicode.com/users'

const isLoading = ref(false)
const successMessage = ref('')
const generalError = ref(null)

// Данные формы.
// reactive - как ref, но для объекта: к полям обращаемся без .value (form.name)
const form = reactive({
  name: '',
  email: '',
  password: '',
  confirmPassword: ''
})

// Ошибки валидации (null - ошибки нет)
const errors = reactive({
  name: null,
  email: null,
  password: null,
  confirmPassword: null
})

// touched - "трогал ли пользователь поле": становится true, когда он ушёл с поля (blur)
const touched = reactive({
  name: false,
  email: false,
  password: false,
  confirmPassword: false
})

// ===== Проверки полей =====
// Каждая функция проверяет одно поле и возвращает текст ошибки или null, если всё хорошо

// Имя: не пустое, длина >= 2
const checkName = () => {
  if (form.name.trim().length < 2) {
    return 'Имя должно быть не короче 2 символов.'
  }
  return null
}

// Email: простой формат что-то@что-то.что-то
const checkEmail = () => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(form.email)) {
    return 'Введите корректный email.'
  }
  return null
}

// Пароль: 8 символов, строчные и заглавные буквы, цифры и спецсимволы.
// /[a-z]/.test(строка) - есть ли в строке хотя бы один символ из квадратных скобок
const checkPassword = () => {
  const password = form.password

  if (password.length < 8) {
    return 'Пароль должен быть не короче 8 символов.'
  }
  if (!/[a-z]/.test(password)) {
    return 'Добавьте строчную латинскую букву (a-z).'
  }
  if (!/[A-Z]/.test(password)) {
    return 'Добавьте заглавную латинскую букву (A-Z).'
  }
  if (!/[0-9]/.test(password)) {
    return 'Добавьте цифру.'
  }
  // ^ внутри [ ] значит "НЕ": ищем символ, который не буква и не цифра - это и есть спецсимвол
  if (!/[^a-zA-Z0-9]/.test(password)) {
    return 'Добавьте спецсимвол (например ! @ # $ %).'
  }
  return null
}

// Подтверждение пароля: не пустое и совпадает с паролем
const checkConfirmPassword = () => {
  if (!form.confirmPassword) {
    return 'Подтвердите пароль.'
  }
  if (form.password !== form.confirmPassword) {
    return 'Пароли не совпадают.'
  }
  return null
}

// Проверить одно поле и записать результат в errors
const validateField = (field) => {
  if (field === 'name') {
    errors.name = checkName()
  } else if (field === 'email') {
    errors.email = checkEmail()
  } else if (field === 'password') {
    errors.password = checkPassword()
    // пароль поменяли - подтверждение тоже надо перепроверить (если его уже вводили)
    if (touched.confirmPassword) {
      errors.confirmPassword = checkConfirmPassword()
    }
  } else if (field === 'confirmPassword') {
    errors.confirmPassword = checkConfirmPassword()
  }
}

// ===== "Живая" валидация =====

// Ушли с поля - отмечаем его как "тронутое" и сразу показываем ошибку
const onBlur = (field) => {
  touched[field] = true
  validateField(field)
}

// Вводим текст - проверяем только "тронутые" поля.
// Так ошибка не выскакивает после первой же буквы,
// но сразу исчезает, как только пользователь её исправил
const onInput = (field) => {
  if (touched[field]) {
    validateField(field)
  }
}

// Проверка всей формы перед отправкой.
// Object.keys(form) - массив названий полей: ['name', 'email', 'password', 'confirmPassword']
const validate = () => {
  Object.keys(form).forEach((field) => {
    touched[field] = true
    validateField(field)
  })

  // форма правильная, только если ни в одном поле нет ошибки
  return !errors.name && !errors.email && !errors.password && !errors.confirmPassword
}

const handleSubmit = (event) => {
  event.preventDefault() // предотвращаем перезагрузку страницы

  successMessage.value = ''
  generalError.value = null

  if (!validate()) {
    return // если есть ошибки валидации - не отправляем
  }

  isLoading.value = true

  // confirmPassword на сервер не отправляем - он нужен только для проверки на странице
  const payload = {
    name: form.name.trim(),
    email: form.email.trim(),
    password: form.password
  }

  // api чисто для примера, т.е. настоящая проверка данных должна быть на бэкенде.
  // axios сам превращает объект в JSON и ставит заголовок Content-Type
  axios
    .post(API_URL, payload)
    .then((response) => {
      // axios сам разбирает ответ из JSON, данные лежат в response.data
      successMessage.value =
        'Регистрация успешно отправлена (демо). Ответ сервера: ' +
        JSON.stringify(response.data, null, 2)
    })
    .catch((err) => {
      if (err.response) {
        // 1) Ошибка сервера: сервер ответил, но с кодом ошибки (4xx или 5xx).
        // axios сам выбрасывает ошибку на такие коды, статус лежит в err.response.status
        const status = err.response.status
        if (status >= 500) {
          generalError.value = `Ошибка на сервере (код ${status}). Попробуйте позже.`
        } else {
          generalError.value = `Сервер отклонил запрос (код ${status}). Проверьте данные.`
        }
      } else if (err.request) {
        // 2) Ошибка сети: запрос ушёл, но ответа нет (нет интернета или сервер недоступен)
        generalError.value = 'Ошибка сети: нет связи с сервером. Проверьте подключение к интернету.'
      } else {
        // 3) Ошибка ещё до отправки запроса
        generalError.value = 'Не удалось отправить запрос: ' + err.message
      }
    })
    .finally(() => {
      // finally выполняется всегда - и при успехе, и при ошибке
      isLoading.value = false
    })
}
</script>

<style scoped>
.form-page {
  max-width: 420px;
  margin: 40px auto;
  padding: 25px 30px;
  border-radius: 10px;
  background-color: white;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

h1 {
  margin-top: 0;
  text-align: center;
}

.form-group {
  /* label, input и ошибка друг под другом */
  display: flex;
  flex-direction: column;
  margin-bottom: 15px;
}

label {
  margin-bottom: 5px;
  font-weight: bold;
}

.form-input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 15px;
}

/* :focus - поле, в котором сейчас курсор */
.form-input:focus {
  border-color: #42b983;
  outline: none;
}

/* поле с ошибкой - красная рамка */
.form-input.input-error {
  border-color: #e53935;
  background-color: #fff5f5;
}

.error-message {
  margin-top: 4px;
  color: #e53935;
  font-size: 13px;
}

/* подсказка под полем пароля */
.hint {
  margin-top: 4px;
  color: #888;
  font-size: 13px;
}

.general-error {
  margin-bottom: 15px;
  padding: 10px;
  border: 1px solid #e57373;
  border-radius: 6px;
  background-color: #ffebee;
  color: #c62828;
}

.btn-submit {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 6px;
  background-color: #42b983;
  color: white;
  font-size: 16px;
  cursor: pointer;
}

.btn-submit:hover {
  background-color: #369870;
}

/* :disabled - кнопка выключена, пока идёт отправка */
.btn-submit:disabled {
  background-color: #aaa;
  cursor: not-allowed;
}

.success-message {
  margin-top: 20px;
  padding: 10px;
  border: 1px solid #81c784;
  border-radius: 6px;
  background-color: #e8f5e9;
  color: #2e7d32;
  /* pre-wrap сохраняет переносы строк и отступы (чтобы JSON был читаемым) */
  white-space: pre-wrap;
  font-size: 14px;
}
</style>
