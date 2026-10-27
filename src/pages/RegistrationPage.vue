<template>
  <div class="form-page">
    <h1>Регистрация</h1>

    <!-- novalidate отключает встроенные проверки браузера: ошибки показываем сами -->
    <form @submit="handleSubmit" class="registration-form" novalidate>
      <!-- Имя -->
      <div class="form-group">
        <!-- for у label совпадает с id у input: клик по подписи ставит курсор в поле -->
        <label for="name">Имя</label>
        <input
          id="name"
          v-model="form.name"
          type="text"
          class="form-input"
          :class="{ 'input-error': errors.name }"
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

// Проверка сложности пароля. Возвращает текст ошибки или null, если пароль подходит.
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

// Простая валидация
const validate = () => {
  // Сброс ошибок. Object.keys(errors) - массив названий полей ['name', 'email', ...]
  Object.keys(errors).forEach((key) => {
    errors[key] = null
  })
  let hasErrors = false

  // Имя: не пустое, длина >= 2
  if (!form.name || form.name.trim().length < 2) {
    errors.name = 'Имя должно быть не короче 2 символов.'
    hasErrors = true
  }

  // Email: не пустой, простой формат что-то@что-то.что-то
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!form.email || !emailRegex.test(form.email)) {
    errors.email = 'Введите корректный email.'
    hasErrors = true
  }

  // Пароль: 8 символов, строчные и заглавные буквы, цифры и спецсимволы
  const passwordError = checkPassword()
  if (passwordError) {
    errors.password = passwordError
    hasErrors = true
  }

  // Подтверждение пароля: совпадает с паролем
  if (form.password !== form.confirmPassword) {
    errors.confirmPassword = 'Пароли не совпадают.'
    hasErrors = true
  } else if (!form.confirmPassword && form.password) {
    // Если пароль заполнен, а подтверждение пустое
    errors.confirmPassword = 'Подтвердите пароль.'
    hasErrors = true
  }

  return !hasErrors
}

const handleSubmit = async (event) => {
  event.preventDefault() // предотвращаем перезагрузку страницы

  successMessage.value = ''
  generalError.value = null

  if (!validate()) {
    return // если есть ошибки валидации - не отправляем
  }

  isLoading.value = true

  try {
    const payload = {
      name: form.name.trim(),
      email: form.email.trim(),
      password: form.password,
      confirmPassword: form.confirmPassword // на бэкенд обычно не отправляют confirmPassword
    }

    // Вариант 1: fetch (нативно)
    // api чисто для примера, т.е. тут должна проводиться проверка на бэке
    const response = await fetch('https://jsonplaceholder.typicode.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    })

    if (!response.ok) {
      throw new Error(`Ошибка сервера: ${response.status}`)
    }

    const data = await response.json()
    successMessage.value =
      'Регистрация успешно отправлена (демо). Данные: ' + JSON.stringify(data, null, 2)
  } catch (err) {
    generalError.value = err instanceof Error ? err.message : 'Произошла ошибка при отправке.'
  } finally {
    isLoading.value = false
  }
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
