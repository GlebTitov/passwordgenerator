<template>
  <div class="min-h-screen bg-gray-50 p-4 md:p-8">
    <!-- Модальное окно мастер-пароля -->
    <div v-if="showMasterPasswordModal" class="fixed inset-0 bg-black bg-opacity-30 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg shadow-xl p-6 max-w-sm w-full mx-4">
        <h2 class="text-xl font-semibold mb-4">🔐 {{ isFirstTime ? 'Создать' : 'Вход' }}</h2>

        <div class="space-y-3">
          <input
              v-model="masterPasswordInput"
              :type="showMasterPassword ? 'text' : 'password'"
              placeholder="Мастер-пароль"
              class="w-full px-3 py-2 border border-gray-300 rounded focus:border-indigo-500 focus:outline-none text-sm"
              @keyup.enter="handleMasterPassword"
          />

          <input
              v-if="isFirstTime"
              v-model="masterPasswordConfirm"
              :type="showMasterPassword ? 'text' : 'password'"
              placeholder="Подтвердите пароль"
              class="w-full px-3 py-2 border border-gray-300 rounded focus:border-indigo-500 focus:outline-none text-sm"
              @keyup.enter="handleMasterPassword"
          />

          <label class="flex items-center gap-2 text-sm cursor-pointer">
            <input v-model="showMasterPassword" type="checkbox" class="w-3 h-3" />
            <span>Показать</span>
          </label>

          <div v-if="masterPasswordError" class="text-xs text-red-600">
            {{ masterPasswordError }}
          </div>

          <button
              @click="handleMasterPassword"
              class="w-full px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700 text-sm"
          >
            {{ isFirstTime ? 'Создать' : 'Войти' }}
          </button>

          <button
              v-if="!isFirstTime"
              @click="skipMasterPassword"
              class="w-full text-gray-500 hover:text-gray-700 text-xs"
          >
            Пропустить
          </button>
        </div>
      </div>
    </div>

    <div class="max-w-5xl mx-auto">
      <!-- Шапка -->
      <div class="flex justify-between items-center mb-6">
        <h1 class="text-2xl md:text-3xl font-bold">Менеджер паролей</h1>
        <button
            v-if="isUnlocked"
            @click="lockVault"
            class="px-3 py-2 text-sm text-gray-600 hover:text-gray-800 border border-gray-300 rounded hover:border-gray-400"
        >
          Выйти
        </button>
      </div>

      <!-- Табы -->
      <div class="bg-white rounded-lg border border-gray-200 mb-6">
        <div class="flex border-b border-gray-200">
          <button
              @click="activeTab = 'generator'"
              :class="['flex-1 px-4 md:px-6 py-3 md:py-4 text-sm font-medium transition', activeTab === 'generator' ? 'text-indigo-600 border-b-2 border-indigo-600' : 'text-gray-500 hover:text-gray-700']"
          >
            🔐 Генератор
          </button>
          <button
              @click="activeTab = 'analysis'"
              :class="['flex-1 px-4 md:px-6 py-3 md:py-4 text-sm font-medium transition', activeTab === 'analysis' ? 'text-indigo-600 border-b-2 border-indigo-600' : 'text-gray-500 hover:text-gray-700']"
              :disabled="!password"
          >
            📊 Анализ
          </button>
          <button
              v-if="isUnlocked"
              @click="activeTab = 'vault'"
              :class="['flex-1 px-4 md:px-6 py-3 md:py-4 text-sm font-medium transition', activeTab === 'vault' ? 'text-indigo-600 border-b-2 border-indigo-600' : 'text-gray-500 hover:text-gray-700']"
          >
            💾 Хранилище ({{ savedPasswords.length }})
          </button>
        </div>
      </div>

      <!-- Контент табов -->
      <div>
        <!-- Таб: Генератор -->
        <div v-show="activeTab === 'generator'" class="space-y-6">
          <div class="bg-white rounded-lg border border-gray-200 p-6">
            <div class="flex items-center gap-3 mb-6">
              <input
                  v-model="password"
                  :type="showPassword ? 'text' : 'password'"
                  :readonly="!isManualInput"
                  class="flex-1 px-4 py-3 border border-gray-300 rounded-lg focus:border-indigo-500 focus:outline-none font-mono text-base"
                  placeholder="Пароль будет здесь"
                  @input="onPasswordInput"
              />
              <button
                  @click="togglePasswordVisibility"
                  class="p-3 hover:bg-gray-100 rounded-lg transition"
                  title="Показать/Скрыть"
              >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path v-if="!showPassword" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                  <path v-if="!showPassword" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
                  <path v-else stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l3.59 3.59m0 0A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.543 7a10.025 10.025 0 01-4.132 5.411m0 0L21 21" />
                </svg>
              </button>
              <button
                  @click="generatePassword"
                  :disabled="isManualInput"
                  :class="['p-3 rounded-lg transition', isManualInput ? 'bg-gray-200 cursor-not-allowed' : 'bg-indigo-600 hover:bg-indigo-700 text-white']"
                  title="Генерировать"
              >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
                </svg>
              </button>
              <button
                  @click="copyPassword"
                  :class="['p-3 rounded-lg transition', copied ? 'bg-green-100 text-green-700' : 'hover:bg-gray-100']"
                  title="Копировать"
              >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path v-if="!copied" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
                  <path v-else stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                </svg>
              </button>
            </div>

            <label class="flex items-center gap-2 text-sm cursor-pointer mb-6">
              <input v-model="isManualInput" type="checkbox" class="w-4 h-4" />
              <span>Ввести пароль вручную</span>
            </label>

            <div v-if="!isManualInput" class="space-y-6">
              <div>
                <div class="flex justify-between text-sm mb-2">
                  <span class="font-medium">Длина пароля</span>
                  <span class="font-semibold text-indigo-600">{{ length }}</span>
                </div>
                <input v-model.number="length" type="range" min="8" max="64" class="w-full h-2" />
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                <label class="flex items-center gap-3 cursor-pointer p-3 border border-gray-200 rounded-lg hover:border-indigo-300 transition">
                  <input v-model="includeUppercase" type="checkbox" class="w-4 h-4" />
                  <span class="text-sm">Заглавные (A-Z)</span>
                </label>
                <label class="flex items-center gap-3 cursor-pointer p-3 border border-gray-200 rounded-lg hover:border-indigo-300 transition">
                  <input v-model="includeLowercase" type="checkbox" class="w-4 h-4" />
                  <span class="text-sm">Строчные (a-z)</span>
                </label>
                <label class="flex items-center gap-3 cursor-pointer p-3 border border-gray-200 rounded-lg hover:border-indigo-300 transition">
                  <input v-model="includeNumbers" type="checkbox" class="w-4 h-4" />
                  <span class="text-sm">Цифры (0-9)</span>
                </label>
                <label class="flex items-center gap-3 cursor-pointer p-3 border border-gray-200 rounded-lg hover:border-indigo-300 transition">
                  <input v-model="includeSymbols" type="checkbox" class="w-4 h-4" />
                  <span class="text-sm">Символы (!@#$)</span>
                </label>
              </div>
            </div>
          </div>

          <!-- Сохранение -->
          <div v-if="password && isUnlocked" class="bg-white rounded-lg border border-gray-200 p-6">
            <h2 class="text-lg font-semibold mb-4">Сохранить пароль</h2>
            <div class="space-y-3">
              <input
                  v-model="passwordLabel"
                  type="text"
                  placeholder="Название (Gmail, ВКонтакте...)"
                  class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:border-indigo-500 focus:outline-none"
              />
              <input
                  v-model="passwordNote"
                  type="text"
                  placeholder="Примечание (необязательно)"
                  class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:border-indigo-500 focus:outline-none"
              />
              <button
                  @click="savePassword"
                  :disabled="!passwordLabel.trim()"
                  class="w-full px-4 py-3 bg-green-600 text-white rounded-lg hover:bg-green-700 disabled:opacity-50 transition"
              >
                Сохранить пароль
              </button>
            </div>
          </div>
        </div>

        <!-- Таб: Анализ -->
        <div v-show="activeTab === 'analysis'" class="space-y-6">
          <div v-if="password" class="bg-white rounded-lg border border-gray-200 p-6">
            <h2 class="text-lg font-semibold mb-6">Анализ надежности</h2>

            <div class="mb-6">
              <div class="flex justify-between items-center mb-2">
                <span class="text-sm font-medium">Надежность</span>
                <span :class="['text-sm font-bold', strengthColor]">{{ strengthLabel }}</span>
              </div>
              <div class="h-3 bg-gray-100 rounded-full overflow-hidden">
                <div :class="strengthBg" :style="{ width: strengthPercent + '%' }" class="h-full transition-all"></div>
              </div>
            </div>

            <div class="grid grid-cols-2 gap-4 mb-6">
              <div class="text-center p-4 bg-gray-50 rounded-lg">
                <div class="text-2xl font-bold text-gray-800">{{ entropy.toFixed(1) }}</div>
                <div class="text-sm text-gray-600 mt-1">Энтропия (бит)</div>
              </div>
              <div class="text-center p-4 bg-gray-50 rounded-lg">
                <div class="text-2xl font-bold text-gray-800">{{ crackTime }}</div>
                <div class="text-sm text-gray-600 mt-1">Время взлома</div>
              </div>
            </div>

            <button
                @click="checkBreach"
                :disabled="checking"
                class="w-full px-4 py-3 bg-orange-500 text-white rounded-lg hover:bg-orange-600 disabled:opacity-50 transition"
            >
              {{ checking ? 'Проверка...' : 'Проверить на утечки' }}
            </button>

            <div v-if="breachCount !== null" class="mt-4 p-4 rounded-lg" :class="breachCount > 0 ? 'bg-red-50 text-red-700' : 'bg-green-50 text-green-700'">
              <div class="flex items-center gap-2">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path v-if="breachCount > 0" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                  <path v-else stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
                <span class="font-medium text-sm">
                  {{ breachCount > 0 ? `Найдено ${breachCount} утечек` : 'Утечек не обнаружено' }}
                </span>
              </div>
            </div>
          </div>

          <div v-else class="bg-white rounded-lg border border-gray-200 p-12 text-center text-gray-400">
            <svg class="w-16 h-16 mx-auto mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
            </svg>
            <p>Сначала сгенерируйте пароль</p>
          </div>
        </div>

        <!-- Таб: Хранилище -->
        <div v-show="activeTab === 'vault'">
          <div v-if="savedPasswords.length > 0" class="bg-white rounded-lg border border-gray-200 p-6">
            <div class="flex justify-between items-center mb-6">
              <h2 class="text-lg font-semibold">Сохраненные пароли ({{ savedPasswords.length }})</h2>
              <button @click="clearAllPasswords" class="text-sm text-red-600 hover:text-red-700">
                Очистить всё
              </button>
            </div>

            <div class="space-y-3">
              <div
                  v-for="(item, index) in savedPasswords"
                  :key="index"
                  class="border border-gray-200 rounded-lg p-4 hover:border-indigo-300 transition"
              >
                <div class="flex justify-between items-start mb-3">
                  <div class="flex-1">
                    <div class="font-medium">{{ item.label }}</div>
                    <div v-if="item.note" class="text-sm text-gray-500 mt-1">{{ item.note }}</div>
                    <div class="text-xs text-gray-400 mt-1">{{ item.createdAt }}</div>
                  </div>
                  <button @click="deletePassword(index)" class="text-red-600 hover:text-red-700 ml-3">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                    </svg>
                  </button>
                </div>
                <div class="flex gap-2">
                  <input
                      :value="item.password"
                      :type="item.visible ? 'text' : 'password'"
                      readonly
                      class="flex-1 px-3 py-2 border border-gray-300 rounded bg-gray-50 font-mono text-sm"
                  />
                  <button @click="togglePasswordVisibilityInList(index)" class="p-2 hover:bg-gray-100 rounded">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                    </svg>
                  </button>
                  <button @click="copyPasswordFromList(item.password)" class="p-2 hover:bg-gray-100 rounded">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
                    </svg>
                  </button>
                </div>
              </div>
            </div>
          </div>

          <div v-else class="bg-white rounded-lg border border-gray-200 p-12 text-center text-gray-400">
            <svg class="w-16 h-16 mx-auto mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
            </svg>
            <p>Нет сохраненных паролей</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import CryptoJS from 'crypto-js';

const activeTab = ref('generator');

const password = ref('');
const length = ref(16);
const includeUppercase = ref(true);
const includeLowercase = ref(true);
const includeNumbers = ref(true);
const includeSymbols = ref(true);
const copied = ref(false);
const breachCount = ref(null);
const checking = ref(false);
const isManualInput = ref(false);
const showPassword = ref(false);

const passwordLabel = ref('');
const passwordNote = ref('');
const savedPasswords = ref([]);

const showMasterPasswordModal = ref(false);
const masterPasswordInput = ref('');
const masterPasswordConfirm = ref('');
const showMasterPassword = ref(false);
const masterPasswordError = ref('');
const isFirstTime = ref(false);
const isUnlocked = ref(false);
const currentMasterPassword = ref('');

const STORAGE_KEY = 'encrypted_passwords';

onMounted(() => {
  checkForExistingData();
});

const checkForExistingData = () => {
  const data = localStorage.getItem(STORAGE_KEY);
  if (data) {
    isFirstTime.value = false;
    showMasterPasswordModal.value = true;
  } else {
    isFirstTime.value = true;
    showMasterPasswordModal.value = true;
  }
};

const encryptData = (data, password) => {
  return CryptoJS.AES.encrypt(JSON.stringify(data), password).toString();
};

const decryptData = (encrypted, password) => {
  try {
    const bytes = CryptoJS.AES.decrypt(encrypted, password);
    const decrypted = bytes.toString(CryptoJS.enc.Utf8);
    if (!decrypted) return null;
    return JSON.parse(decrypted);
  } catch (error) {
    return null;
  }
};

const handleMasterPassword = () => {
  masterPasswordError.value = '';

  if (isFirstTime.value) {
    if (masterPasswordInput.value.length < 8) {
      masterPasswordError.value = 'Мастер-пароль должен быть не менее 8 символов';
      return;
    }
    if (masterPasswordInput.value !== masterPasswordConfirm.value) {
      masterPasswordError.value = 'Пароли не совпадают';
      return;
    }
    currentMasterPassword.value = masterPasswordInput.value;
    isUnlocked.value = true;
    showMasterPasswordModal.value = false;
    savedPasswords.value = [];
    saveToStorage();
  } else {
    const encrypted = localStorage.getItem(STORAGE_KEY);
    const decrypted = decryptData(encrypted, masterPasswordInput.value);

    if (decrypted) {
      currentMasterPassword.value = masterPasswordInput.value;
      savedPasswords.value = decrypted;
      isUnlocked.value = true;
      showMasterPasswordModal.value = false;
    } else {
      masterPasswordError.value = 'Неверный мастер-пароль';
    }
  }

  masterPasswordInput.value = '';
  masterPasswordConfirm.value = '';
};

const skipMasterPassword = () => {
  showMasterPasswordModal.value = false;
  isUnlocked.value = false;
};

const lockVault = () => {
  if (confirm('Выйти и заблокировать хранилище?')) {
    currentMasterPassword.value = '';
    isUnlocked.value = false;
    savedPasswords.value = [];
    showMasterPasswordModal.value = true;
    isFirstTime.value = false;
    activeTab.value = 'generator';
  }
};

const saveToStorage = () => {
  if (!isUnlocked.value || !currentMasterPassword.value) return;
  const encrypted = encryptData(savedPasswords.value, currentMasterPassword.value);
  localStorage.setItem(STORAGE_KEY, encrypted);
};

const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value;
};

const onPasswordInput = () => {
  breachCount.value = null;
};

const generatePassword = () => {
  let charset = '';
  if (includeUppercase.value) charset += 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  if (includeLowercase.value) charset += 'abcdefghijklmnopqrstuvwxyz';
  if (includeNumbers.value) charset += '0123456789';
  if (includeSymbols.value) charset += '!@#$%^&*()_+-=[]{}|;:,.<>?';

  if (charset === '') {
    password.value = '';
    return;
  }

  let newPassword = '';
  const array = new Uint8Array(length.value);
  crypto.getRandomValues(array);

  for (let i = 0; i < length.value; i++) {
    newPassword += charset[array[i] % charset.length];
  }

  password.value = newPassword;
  breachCount.value = null;
};

const savePassword = () => {
  if (!passwordLabel.value.trim() || !password.value || !isUnlocked.value) return;

  const newPassword = {
    label: passwordLabel.value.trim(),
    note: passwordNote.value.trim(),
    password: password.value,
    createdAt: new Date().toLocaleString('ru-RU'),
    visible: false
  };

  savedPasswords.value.push(newPassword);
  saveToStorage();

  passwordLabel.value = '';
  passwordNote.value = '';

  alert('Пароль сохранен!');
  activeTab.value = 'vault';
};

const deletePassword = (index) => {
  if (confirm('Удалить пароль?')) {
    savedPasswords.value.splice(index, 1);
    saveToStorage();
  }
};

const clearAllPasswords = () => {
  if (confirm('Удалить все пароли?')) {
    savedPasswords.value = [];
    saveToStorage();
  }
};

const togglePasswordVisibilityInList = (index) => {
  savedPasswords.value[index].visible = !savedPasswords.value[index].visible;
};

const copyPasswordFromList = async (pwd) => {
  try {
    await navigator.clipboard.writeText(pwd);
    alert('Скопировано!');
  } catch (err) {
    console.error('Ошибка:', err);
  }
};

const copyPassword = async () => {
  if (!password.value) return;

  try {
    await navigator.clipboard.writeText(password.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    console.error('Ошибка:', err);
  }
};

const entropy = computed(() => {
  if (!password.value) return 0;

  let charsetSize = 0;
  if (/[A-Z]/.test(password.value)) charsetSize += 26;
  if (/[a-z]/.test(password.value)) charsetSize += 26;
  if (/[0-9]/.test(password.value)) charsetSize += 10;
  if (/[^A-Za-z0-9]/.test(password.value)) charsetSize += 32;

  return Math.log2(Math.pow(charsetSize, password.value.length));
});

const strengthScore = computed(() => {
  const e = entropy.value;
  if (e < 28) return 0;
  if (e < 36) return 1;
  if (e < 60) return 2;
  if (e < 128) return 3;
  return 4;
});

const strengthLabel = computed(() => {
  const labels = ['Очень слабый', 'Слабый', 'Средний', 'Сильный', 'Очень сильный'];
  return labels[strengthScore.value];
});

const strengthColor = computed(() => {
  const colors = ['text-red-600', 'text-orange-600', 'text-yellow-600', 'text-green-600', 'text-emerald-600'];
  return colors[strengthScore.value];
});

const strengthBg = computed(() => {
  const colors = ['bg-red-500', 'bg-orange-500', 'bg-yellow-500', 'bg-green-500', 'bg-emerald-500'];
  return colors[strengthScore.value];
});

const strengthPercent = computed(() => {
  return (strengthScore.value + 1) * 20;
});

const crackTime = computed(() => {
  if (!password.value) return '—';

  const combinations = Math.pow(2, entropy.value);
  const attemptsPerSecond = 1e9;
  const seconds = combinations / attemptsPerSecond / 2;

  if (seconds < 1) return '< 1 сек';
  if (seconds < 60) return `${Math.round(seconds)} сек`;
  if (seconds < 3600) return `${Math.round(seconds / 60)} мин`;
  if (seconds < 86400) return `${Math.round(seconds / 3600)} ч`;
  if (seconds < 31536000) return `${Math.round(seconds / 86400)} дн`;
  if (seconds < 31536000000) return `${Math.round(seconds / 31536000)} лет`;

  return `${(seconds / 31536000).toExponential(2)} лет`;
});

const checkBreach = async () => {
  if (!password.value) return;

  checking.value = true;
  breachCount.value = null;

  try {
    const encoder = new TextEncoder();
    const data = encoder.encode(password.value);
    const hashBuffer = await crypto.subtle.digest('SHA-1', data);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('').toUpperCase();

    const prefix = hashHex.substring(0, 5);
    const suffix = hashHex.substring(5);

    const response = await fetch(`https://api.pwnedpasswords.com/range/${prefix}`);
    const text = await response.text();

    const lines = text.split('\n');
    const found = lines.find(line => line.startsWith(suffix));

    if (found) {
      const count = parseInt(found.split(':')[1]);
      breachCount.value = count;
    } else {
      breachCount.value = 0;
    }
  } catch (error) {
    console.error('Ошибка:', error);
    alert('Ошибка проверки');
  } finally {
    checking.value = false;
  }
};

generatePassword();

watch([length, includeUppercase, includeLowercase, includeNumbers, includeSymbols], () => {
  breachCount.value = null;
});
</script>

<style scoped>
input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  background: #e5e7eb;
  border-radius: 4px;
  cursor: pointer;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #4f46e5;
  cursor: pointer;
}

input[type="range"]::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #4f46e5;
  cursor: pointer;
  border: none;
}

input[type="checkbox"] {
  cursor: pointer;
  accent-color: #4f46e5;
}
</style>