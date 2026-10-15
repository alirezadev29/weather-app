<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import OutfitIcon from './components/OutfitIcon.vue';

// ============================================
// State
// ============================================

const city = ref('');
const weather = ref(null);
const forecast = ref([]);
const loading = ref(false);
const error = ref('');
const searchHistory = ref([]);

// حالت مقایسه
const compareMode = ref(false);
const city1 = ref('');
const city2 = ref('');
const weather1 = ref(null);
const weather2 = ref(null);
const compareLoading = ref(false);
const compareError = ref('');

const MAX_HISTORY = 8;
const STORAGE_KEY = 'weather-history';

// ============================================
// Computed - Weather اصلی
// ============================================

const weatherType = computed(() => {
  if (!weather.value) return 'default';
  return getWeatherType(weather.value.code);
});

const weatherText = computed(() => {
  if (!weather.value) return '';
  return getWeatherText(weather.value.code);
});

const outfitSuggestion = computed(() => {
  if (!weather.value) return null;
  return getOutfitSuggestion(weather.value);
});

// ============================================
// Computed - مقایسه
// ============================================

const tempDiff = computed(() => {
  if (!weather1.value || !weather2.value) return 0;
  return Math.abs(Math.round(weather1.value.temp - weather2.value.temp));
});

const warmerCity = computed(() => {
  if (!weather1.value || !weather2.value) return null;
  return weather1.value.temp > weather2.value.temp ? weather1.value : weather2.value;
});

const windierCity = computed(() => {
  if (!weather1.value || !weather2.value) return null;
  return weather1.value.wind > weather2.value.wind ? weather1.value : weather2.value;
});

// ============================================
// توابع کمکی
// ============================================

function getWeatherText(code) {
  const codes = {
    0: 'آسمان صاف', 1: 'عمدتاً صاف', 2: 'نیمه ابری', 3: 'ابری',
    45: 'مه', 48: 'مه یخ‌زده',
    51: 'نم‌نم باران', 53: 'باران سبک', 55: 'باران متوسط',
    61: 'باران کم', 63: 'باران', 65: 'باران شدید',
    71: 'برف کم', 73: 'برف', 75: 'برف شدید',
    80: 'رگبار کم', 81: 'رگبار', 82: 'رگبار شدید',
    95: 'رعد و برق', 96: 'رعد و برق با تگرگ', 99: 'طوفان شدید'
  };
  return codes[code] || 'نامشخص';
}

function getWeatherType(code) {
  if (code === 0 || code === 1) return 'clear';
  if (code === 2 || code === 3) return 'cloudy';
  if (code >= 45 && code <= 48) return 'fog';
  if (code >= 51 && code <= 67) return 'rain';
  if (code >= 71 && code <= 77) return 'snow';
  if (code >= 80 && code <= 82) return 'rain';
  if (code >= 95) return 'storm';
  return 'clear';
}

function getOutfitSuggestion(w) {
  const temp = w.temp;
  const code = w.code;
  const wind = w.wind;

  const items = [];
  const tips = [];

  if (temp >= 30) {
    items.push(
      { icon: 'tshirt', label: 'تی‌شرت نازک' },
      { icon: 'shorts', label: 'شلوارک' },
      { icon: 'sunglasses', label: 'عینک آفتابی' },
      { icon: 'hat', label: 'کلاه لبه‌دار' }
    );
    tips.push('آب زیاد بنوش 💧');
  } else if (temp >= 25) {
    items.push(
      { icon: 'tshirt', label: 'تی‌شرت' },
      { icon: 'shorts', label: 'شلوارک یا شلوار نازک' },
      { icon: 'sneakers', label: 'کفش راحت' }
    );
  } else if (temp >= 20) {
    items.push(
      { icon: 'tshirt', label: 'تی‌شرت یا پیراهن' },
      { icon: 'pants', label: 'شلوار راحت' },
      { icon: 'sneakers', label: 'کفش راحت' }
    );
  } else if (temp >= 15) {
    items.push(
      { icon: 'shirt', label: 'پیراهن یا تی‌شرت' },
      { icon: 'jacket', label: 'کاپشن سبک' },
      { icon: 'pants', label: 'شلوار بلند' },
      { icon: 'sneakers', label: 'کفش بسته' }
    );
  } else if (temp >= 10) {
    items.push(
      { icon: 'shirt', label: 'پیراهن آستین بلند' },
      { icon: 'jacket', label: 'کاپشن' },
      { icon: 'pants', label: 'شلوار بلند' },
      { icon: 'sneakers', label: 'کفش بسته' }
    );
  } else if (temp >= 5) {
    items.push(
      { icon: 'sweater', label: 'سویشرت ضخیم' },
      { icon: 'coat', label: 'پالتو' },
      { icon: 'pants', label: 'شلوار ضخیم' },
      { icon: 'scarf', label: 'شال گردن' }
    );
  } else if (temp >= 0) {
    items.push(
      { icon: 'sweater', label: 'سویشرت گرم' },
      { icon: 'coat', label: 'پالتو زمستانی' },
      { icon: 'pants', label: 'شلوار ضخیم' },
      { icon: 'scarf', label: 'شال گردن' },
      { icon: 'gloves', label: 'دستکش' },
      { icon: 'boots', label: 'چکمه' }
    );
    tips.push('لباس چند لایه بپوش 🧥');
  } else {
    items.push(
      { icon: 'thermal', label: 'لباس زیر گرم' },
      { icon: 'sweater', label: 'سویشرت ضخیم' },
      { icon: 'coat', label: 'پالتو زمستانی' },
      { icon: 'scarf', label: 'شال گردن' },
      { icon: 'gloves', label: 'دستکش' },
      { icon: 'boots', label: 'چکمه گرم' }
    );
    tips.push('از سرمازدگی مراقبت کن ❄️');
  }

  if ((code >= 51 && code <= 67) || (code >= 80 && code <= 82)) {
    items.push({ icon: 'umbrella', label: 'چتر' });
    tips.push('چتر یادت نره! ☔');
  }
  if (code >= 71 && code <= 77) {
    items.push({ icon: 'umbrella', label: 'چتر برفی' });
  }
  if (code >= 95) {
    items.push({ icon: 'umbrella', label: 'چتر محکم' });
    tips.push('اگه می‌تونی در خونه بمون ⛈');
  }
  if (wind >= 30) {
    tips.push('کاپشن ضد باد بپوش 💨');
  }

  return { items, tips };
}

function formatDate(dateString, index) {
  if (index === 0) return 'امروز';
  if (index === 1) return 'فردا';
  const date = new Date(dateString);
  const days = ['یکشنبه', 'دوشنبه', 'سه‌شنبه', 'چهارشنبه', 'پنجشنبه', 'جمعه', 'شنبه'];
  return days[date.getDay()];
}

// ============================================
// LocalStorage
// ============================================

function loadHistory() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    searchHistory.value = saved ? JSON.parse(saved) : [];
  } catch (err) {
    searchHistory.value = [];
  }
}

watch(
  searchHistory,
  (newHistory) => {
    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(newHistory));
    } catch (err) {}
  },
  { deep: true }
);

// ============================================
// توابع تاریخچه
// ============================================

function addToHistory(cityName) {
  searchHistory.value = searchHistory.value.filter(
    (item) => item.toLowerCase() !== cityName.toLowerCase()
  );
  searchHistory.value.unshift(cityName);
  if (searchHistory.value.length > MAX_HISTORY) {
    searchHistory.value = searchHistory.value.slice(0, MAX_HISTORY);
  }
}

function removeFromHistory(cityName) {
  searchHistory.value = searchHistory.value.filter((item) => item !== cityName);
}

function clearHistory() {
  searchHistory.value = [];
}

function searchFromHistory(cityName) {
  city.value = cityName;
  search();
}

// ============================================
// API Helper
// ============================================

async function fetchWeather(cityName) {
  const geoUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${cityName}&count=1&language=fa&format=json`;
  const geoResponse = await fetch(geoUrl);
  const geoData = await geoResponse.json();

  if (!geoData.results || geoData.results.length === 0) {
    throw new Error(`شهر "${cityName}" پیدا نشد`);
  }

  const { latitude, longitude, name, country } = geoData.results[0];

  const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true&daily=weathercode,temperature_2m_max,temperature_2m_min&timezone=auto&forecast_days=6`;
  const weatherResponse = await fetch(weatherUrl);
  const weatherData = await weatherResponse.json();

  const forecastList = [];
  for (let i = 1; i <= 5; i++) {
    forecastList.push({
      date: weatherData.daily.time[i],
      code: weatherData.daily.weathercode[i],
      tempMax: weatherData.daily.temperature_2m_max[i],
      tempMin: weatherData.daily.temperature_2m_min[i],
      type: getWeatherType(weatherData.daily.weathercode[i])
    });
  }

  return {
    weather: {
      city: name,
      country: country,
      temp: weatherData.current_weather.temperature,
      wind: weatherData.current_weather.windspeed,
      code: weatherData.current_weather.weathercode
    },
    forecast: forecastList
  };
}

// ============================================
// جستجوی عادی
// ============================================

async function search() {
  if (city.value.trim() === '') return;

  loading.value = true;
  error.value = '';
  weather.value = null;
  forecast.value = [];

  try {
    const result = await fetchWeather(city.value);
    weather.value = result.weather;
    forecast.value = result.forecast;
    addToHistory(result.weather.city);
    city.value = '';
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
}

// ============================================
// مقایسه دو شهر
// ============================================

async function compare() {
  if (!city1.value.trim() || !city2.value.trim()) {
    compareError.value = 'هر دو شهر را وارد کنید';
    return;
  }

  compareLoading.value = true;
  compareError.value = '';
  weather1.value = null;
  weather2.value = null;

  try {
    // ⭐ Promise.all — دو API به صورت موازی
    const [result1, result2] = await Promise.all([
      fetchWeather(city1.value),
      fetchWeather(city2.value)
    ]);

    weather1.value = result1.weather;
    weather2.value = result2.weather;

    // افزودن به تاریخچه
    addToHistory(result1.weather.city);
    addToHistory(result2.weather.city);
  } catch (err) {
    compareError.value = err.message;
  } finally {
    compareLoading.value = false;
  }
}

function swapCities() {
  const temp = city1.value;
  city1.value = city2.value;
  city2.value = temp;

  if (weather1.value && weather2.value) {
    const tempW = weather1.value;
    weather1.value = weather2.value;
    weather2.value = tempW;
  }
}

function toggleCompareMode() {
  compareMode.value = !compareMode.value;
  // پاک کردن داده‌های قبلی
  weather.value = null;
  forecast.value = [];
  weather1.value = null;
  weather2.value = null;
  error.value = '';
  compareError.value = '';
}

// ============================================
// Lifecycle
// ============================================

onMounted(() => {
  loadHistory();
});
</script>

<template>
  <div class="app" :class="`bg-${weatherType}`">
    <!-- لایه‌های تزئینی -->
    <div class="bg-layer">
      <div class="bubble bubble-1"></div>
      <div class="bubble bubble-2"></div>
      <div class="bubble bubble-3"></div>
      <div class="bubble bubble-4"></div>
      <div class="bubble bubble-5"></div>
      <div class="glow glow-1"></div>
      <div class="glow glow-2"></div>
      <div class="glow glow-3"></div>
    </div>

    <!-- محتوای اصلی -->
    <div class="container">
      <h1 class="title">
        <span class="icon-title">🌤</span>
        اپ آب و هوا
      </h1>

      <!-- دکمه‌های حالت -->
      <div class="mode-tabs">
        <button
          :class="['mode-tab', { active: !compareMode }]"
          @click="compareMode && toggleCompareMode()"
        >
          🔍 جستجو
        </button>
        <button
          :class="['mode-tab', { active: compareMode }]"
          @click="!compareMode && toggleCompareMode()"
        >
          ⚖️ مقایسه
        </button>
      </div>

      <!-- ============ حالت جستجو ============ -->
      <template v-if="!compareMode">
        <div class="search-box">
          <input
            v-model="city"
            type="text"
            placeholder="اسم شهر را وارد کن..."
            @keydown.enter="search"
          />
          <button @click="search" :disabled="loading">
            {{ loading ? '...' : '🔍 جستجو' }}
          </button>
        </div>

        <!-- تاریخچه -->
        <div v-if="searchHistory.length > 0" class="history-section">
          <div class="history-header">
            <span class="history-title">📍 جستجوهای اخیر</span>
            <button class="clear-btn" @click="clearHistory">پاک کردن همه</button>
          </div>
          <div class="history-list">
            <button
              v-for="item in searchHistory"
              :key="item"
              class="history-item"
              @click="searchFromHistory(item)"
            >
              <span class="history-city">{{ item }}</span>
              <span
                class="history-remove"
                @click.stop="removeFromHistory(item)"
              >
                ✕
              </span>
            </button>
          </div>
        </div>

        <!-- Loading -->
        <div v-if="loading" class="loading">
          <div class="spinner"></div>
          <p>در حال دریافت...</p>
        </div>

        <!-- Error -->
        <div v-else-if="error" class="error">
          <p>❌ {{ error }}</p>
        </div>

        <!-- Weather -->
        <template v-else-if="weather">
          <div class="weather-card">
            <h2 class="city-name">
              {{ weather.city }}
              <small v-if="weather.country">, {{ weather.country }}</small>
            </h2>

            <div class="weather-icon">
              <svg v-if="weatherType === 'clear'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <circle cx="50" cy="50" r="20" fill="#FFB800"/>
                <g stroke="#FFB800" stroke-width="3" stroke-linecap="round">
                  <line x1="50" y1="10" x2="50" y2="22"/>
                  <line x1="50" y1="78" x2="50" y2="90"/>
                  <line x1="10" y1="50" x2="22" y2="50"/>
                  <line x1="78" y1="50" x2="90" y2="50"/>
                </g>
              </svg>
              <svg v-else-if="weatherType === 'cloudy'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <circle cx="65" cy="40" r="18" fill="#FFB800" opacity="0.7"/>
                <path d="M25 65 Q15 65 15 55 Q15 45 25 45 Q27 32 42 32 Q55 32 58 45 Q72 45 72 57 Q72 68 60 68 L25 65 Z" fill="#E5E7EB"/>
              </svg>
              <svg v-else-if="weatherType === 'rain'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <path d="M20 50 Q15 50 15 42 Q15 34 25 34 Q28 24 40 24 Q52 24 55 34 Q70 34 70 45 Q70 55 60 55 L20 50 Z" fill="#9CA3AF"/>
                <g stroke="#3B82F6" stroke-width="2.5" stroke-linecap="round">
                  <line x1="30" y1="60" x2="28" y2="70"/>
                  <line x1="45" y1="62" x2="43" y2="72"/>
                  <line x1="60" y1="60" x2="58" y2="70"/>
                </g>
              </svg>
              <svg v-else-if="weatherType === 'snow'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <path d="M20 45 Q15 45 15 37 Q15 29 25 29 Q28 19 40 19 Q52 19 55 29 Q70 29 70 40 Q70 50 60 50 L20 45 Z" fill="#9CA3AF"/>
                <g fill="#60A5FA">
                  <circle cx="30" cy="62" r="2.5"/>
                  <circle cx="45" cy="68" r="2.5"/>
                  <circle cx="60" cy="62" r="2.5"/>
                </g>
              </svg>
              <svg v-else-if="weatherType === 'storm'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <path d="M20 45 Q15 45 15 37 Q15 29 25 29 Q28 19 40 19 Q52 19 55 29 Q70 29 70 40 Q70 50 60 50 L20 45 Z" fill="#4B5563"/>
                <polygon points="48,50 42,68 50,68 44,85 58,64 50,64 56,50" fill="#FBBF24"/>
              </svg>
              <svg v-else viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <g stroke="#9CA3AF" stroke-width="4" stroke-linecap="round">
                  <line x1="20" y1="40" x2="80" y2="40"/>
                  <line x1="25" y1="52" x2="75" y2="52"/>
                  <line x1="20" y1="64" x2="80" y2="64"/>
                </g>
              </svg>
            </div>

            <div class="temp">{{ Math.round(weather.temp) }}°C</div>
            <p class="description">{{ weatherText }}</p>

            <div class="details">
              <div class="detail-item">
                <span>💨 باد</span>
                <strong>{{ weather.wind }} km/h</strong>
              </div>
              <div class="detail-item">
                <span>🌡 دما</span>
                <strong>{{ Math.round(weather.temp) }}°C</strong>
              </div>
              <div class="detail-item">
                <span>🌍 کشور</span>
                <strong>{{ weather.country }}</strong>
              </div>
            </div>
          </div>

          <!-- پیش‌بینی -->
          <div v-if="forecast.length > 0" class="forecast-section">
            <h3 class="forecast-title">📅 پیش‌بینی ۵ روزه</h3>
            <div class="forecast-grid">
              <div
                v-for="(day, index) in forecast"
                :key="day.date"
                class="forecast-card"
              >
                <span class="forecast-day">{{ formatDate(day.date, index + 1) }}</span>
                <div class="forecast-icon">
                  <svg v-if="day.type === 'clear'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <circle cx="50" cy="50" r="18" fill="#FFB800"/>
                    <g stroke="#FFB800" stroke-width="3" stroke-linecap="round">
                      <line x1="50" y1="14" x2="50" y2="24"/>
                      <line x1="50" y1="76" x2="50" y2="86"/>
                      <line x1="14" y1="50" x2="24" y2="50"/>
                      <line x1="76" y1="50" x2="86" y2="50"/>
                    </g>
                  </svg>
                  <svg v-else-if="day.type === 'cloudy'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <circle cx="62" cy="42" r="15" fill="#FFB800" opacity="0.7"/>
                    <path d="M28 68 Q18 68 18 58 Q18 48 28 48 Q30 36 44 36 Q56 36 58 48 Q70 48 70 60 Q70 70 60 70 L28 68 Z" fill="#E5E7EB"/>
                  </svg>
                  <svg v-else-if="day.type === 'rain'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <path d="M22 52 Q16 52 16 44 Q16 36 26 36 Q29 26 42 26 Q54 26 56 36 Q70 36 70 47 Q70 56 60 56 L22 52 Z" fill="#9CA3AF"/>
                    <g stroke="#3B82F6" stroke-width="2.5" stroke-linecap="round">
                      <line x1="32" y1="62" x2="30" y2="72"/>
                      <line x1="48" y1="64" x2="46" y2="74"/>
                      <line x1="62" y1="62" x2="60" y2="72"/>
                    </g>
                  </svg>
                  <svg v-else-if="day.type === 'snow'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <path d="M22 48 Q16 48 16 40 Q16 32 26 32 Q29 22 42 22 Q54 22 56 32 Q70 32 70 42 Q70 52 60 52 L22 48 Z" fill="#9CA3AF"/>
                    <g fill="#60A5FA">
                      <circle cx="32" cy="64" r="2.5"/>
                      <circle cx="48" cy="70" r="2.5"/>
                      <circle cx="62" cy="64" r="2.5"/>
                    </g>
                  </svg>
                  <svg v-else-if="day.type === 'storm'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <path d="M22 48 Q16 48 16 40 Q16 32 26 32 Q29 22 42 22 Q54 22 56 32 Q70 32 70 42 Q70 52 60 52 L22 48 Z" fill="#4B5563"/>
                    <polygon points="48,52 42,68 50,68 44,84 58,64 50,64 56,52" fill="#FBBF24"/>
                  </svg>
                  <svg v-else viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <g stroke="#9CA3AF" stroke-width="4" stroke-linecap="round">
                      <line x1="22" y1="44" x2="78" y2="44"/>
                      <line x1="26" y1="54" x2="74" y2="54"/>
                      <line x1="22" y1="64" x2="78" y2="64"/>
                    </g>
                  </svg>
                </div>
                <div class="forecast-temps">
                  <span class="temp-max">{{ Math.round(day.tempMax) }}°</span>
                  <span class="temp-min">{{ Math.round(day.tempMin) }}°</span>
                </div>
              </div>
            </div>
          </div>

          <!-- پیشنهاد لباس -->
          <div v-if="outfitSuggestion" class="outfit-section">
            <h3 class="outfit-title">👕 پیشنهاد لباس برای امروز</h3>
            <div class="outfit-grid">
              <div
                v-for="(item, index) in outfitSuggestion.items"
                :key="index"
                class="outfit-item"
              >
                <div class="outfit-icon-wrapper">
                  <OutfitIcon :type="item.icon" :size="40" />
                </div>
                <span class="outfit-label">{{ item.label }}</span>
              </div>
            </div>
            <div v-if="outfitSuggestion.tips.length > 0" class="outfit-tips">
              <div
                v-for="(tip, index) in outfitSuggestion.tips"
                :key="index"
                class="outfit-tip"
              >
                {{ tip }}
              </div>
            </div>
          </div>
        </template>

        <!-- Empty -->
        <div v-else class="empty">
          <div class="empty-icon">🌍</div>
          <p>هنوز شهری جستجو نکردی</p>
          <span class="empty-hint">مثلاً: تهران، لندن، مسکو</span>
        </div>
      </template>

      <!-- ============ حالت مقایسه ============ -->
      <template v-else>
        <div class="compare-inputs">
          <div class="compare-input-group">
            <label>شهر اول</label>
            <input
              v-model="city1"
              type="text"
              placeholder="مثلاً تهران"
              @keydown.enter="compare"
            />
          </div>

          <button class="swap-btn" @click="swapCities" title="تعویض">
            ⇄
          </button>

          <div class="compare-input-group">
            <label>شهر دوم</label>
            <input
              v-model="city2"
              type="text"
              placeholder="مثلاً لندن"
              @keydown.enter="compare"
            />
          </div>
        </div>

        <button class="compare-btn" @click="compare" :disabled="compareLoading">
          {{ compareLoading ? 'در حال مقایسه...' : '⚖️ مقایسه کن' }}
        </button>

        <!-- Loading -->
        <div v-if="compareLoading" class="loading">
          <div class="spinner"></div>
          <p>در حال مقایسه...</p>
        </div>

        <!-- Error -->
        <div v-else-if="compareError" class="error">
          <p>❌ {{ compareError }}</p>
        </div>

        <!-- نتایج مقایسه -->
        <div v-else-if="weather1 && weather2" class="compare-results">
          <div class="compare-grid">
            <!-- شهر ۱ -->
            <div class="compare-card">
              <h3 class="compare-city">{{ weather1.city }}</h3>
              <div class="compare-icon">
                <svg v-if="getWeatherType(weather1.code) === 'clear'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <circle cx="50" cy="50" r="20" fill="#FFB800"/>
                </svg>
                <svg v-else-if="getWeatherType(weather1.code) === 'cloudy'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <path d="M25 65 Q15 65 15 55 Q15 45 25 45 Q27 32 42 32 Q55 32 58 45 Q72 45 72 57 Q72 68 60 68 L25 65 Z" fill="#E5E7EB"/>
                </svg>
                <svg v-else-if="getWeatherType(weather1.code) === 'rain'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <path d="M20 50 Q15 50 15 42 Q15 34 25 34 Q28 24 40 24 Q52 24 55 34 Q70 34 70 45 Q70 55 60 55 L20 50 Z" fill="#9CA3AF"/>
                  <g stroke="#3B82F6" stroke-width="2.5" stroke-linecap="round">
                    <line x1="30" y1="60" x2="28" y2="70"/>
                    <line x1="50" y1="62" x2="48" y2="72"/>
                  </g>
                </svg>
                <svg v-else-if="getWeatherType(weather1.code) === 'snow'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <path d="M20 45 Q15 45 15 37 Q15 29 25 29 Q28 19 40 19 Q52 19 55 29 Q70 29 70 40 Q70 50 60 50 L20 45 Z" fill="#9CA3AF"/>
                  <g fill="#60A5FA">
                    <circle cx="35" cy="65" r="3"/>
                    <circle cx="55" cy="70" r="3"/>
                  </g>
                </svg>
                <svg v-else viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <g stroke="#9CA3AF" stroke-width="4" stroke-linecap="round">
                    <line x1="20" y1="45" x2="80" y2="45"/>
                    <line x1="25" y1="58" x2="75" y2="58"/>
                  </g>
                </svg>
              </div>
              <div class="compare-temp">{{ Math.round(weather1.temp) }}°C</div>
              <p class="compare-desc">{{ getWeatherText(weather1.code) }}</p>
              <div class="compare-details">
                <div><span>💨</span> {{ weather1.wind }} km/h</div>
                <div><span>🌍</span> {{ weather1.country }}</div>
              </div>
            </div>

            <!-- VS -->
            <div class="vs-badge">VS</div>

            <!-- شهر ۲ -->
            <div class="compare-card">
              <h3 class="compare-city">{{ weather2.city }}</h3>
              <div class="compare-icon">
                <svg v-if="getWeatherType(weather2.code) === 'clear'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <circle cx="50" cy="50" r="20" fill="#FFB800"/>
                </svg>
                <svg v-else-if="getWeatherType(weather2.code) === 'cloudy'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <path d="M25 65 Q15 65 15 55 Q15 45 25 45 Q27 32 42 32 Q55 32 58 45 Q72 45 72 57 Q72 68 60 68 L25 65 Z" fill="#E5E7EB"/>
                </svg>
                <svg v-else-if="getWeatherType(weather2.code) === 'rain'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <path d="M20 50 Q15 50 15 42 Q15 34 25 34 Q28 24 40 24 Q52 24 55 34 Q70 34 70 45 Q70 55 60 55 L20 50 Z" fill="#9CA3AF"/>
                  <g stroke="#3B82F6" stroke-width="2.5" stroke-linecap="round">
                    <line x1="30" y1="60" x2="28" y2="70"/>
                    <line x1="50" y1="62" x2="48" y2="72"/>
                  </g>
                </svg>
                <svg v-else-if="getWeatherType(weather2.code) === 'snow'" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <path d="M20 45 Q15 45 15 37 Q15 29 25 29 Q28 19 40 19 Q52 19 55 29 Q70 29 70 40 Q70 50 60 50 L20 45 Z" fill="#9CA3AF"/>
                  <g fill="#60A5FA">
                    <circle cx="35" cy="65" r="3"/>
                    <circle cx="55" cy="70" r="3"/>
                  </g>
                </svg>
                <svg v-else viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                  <g stroke="#9CA3AF" stroke-width="4" stroke-linecap="round">
                    <line x1="20" y1="45" x2="80" y2="45"/>
                    <line x1="25" y1="58" x2="75" y2="58"/>
                  </g>
                </svg>
              </div>
              <div class="compare-temp">{{ Math.round(weather2.temp) }}°C</div>
              <p class="compare-desc">{{ getWeatherText(weather2.code) }}</p>
              <div class="compare-details">
                <div><span>💨</span> {{ weather2.wind }} km/h</div>
                <div><span>🌍</span> {{ weather2.country }}</div>
              </div>
            </div>
          </div>

          <!-- خلاصه مقایسه -->
          <div class="compare-summary">
            <h3 class="summary-title">📊 خلاصه مقایسه</h3>

            <div class="summary-item">
              <span class="summary-label">🌡 اختلاف دما</span>
              <strong class="summary-value">{{ tempDiff }} درجه</strong>
            </div>

            <div class="summary-item">
              <span class="summary-label">🔥 گرم‌تر</span>
              <strong class="summary-value">{{ warmerCity.city }}</strong>
            </div>

            <div class="summary-item">
              <span class="summary-label">💨 باد‌دارتر</span>
              <strong class="summary-value">{{ windierCity.city }}</strong>
            </div>
          </div>
        </div>

        <!-- Empty مقایسه -->
        <div v-else class="empty">
          <div class="empty-icon">⚖️</div>
          <p>دو شهر رو وارد کن تا مقایسه کنم</p>
          <span class="empty-hint">مثلاً: تهران و لندن</span>
        </div>
      </template>
    </div>
  </div>
</template>

<style scoped>
/* ============ APP ============ */
.app {
  position: relative;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  overflow: hidden;
  transition: background 1.2s ease;
}

.app.bg-default {
  background: linear-gradient(-45deg, #1a7a5c, #2c3e50, #42b883, #145a44);
  background-size: 400% 400%;
  animation: gradientMove 15s ease infinite;
}

.app.bg-clear {
  background: linear-gradient(-45deg, #FFB800, #FFD966, #FFA500, #FFC93C);
  background-size: 400% 400%;
  animation: gradientMove 15s ease infinite;
}

.app.bg-cloudy {
  background: linear-gradient(-45deg, #4B5563, #9CA3AF, #6B7280, #D1D5DB);
  background-size: 400% 400%;
  animation: gradientMove 15s ease infinite;
}

.app.bg-rain {
  background: linear-gradient(-45deg, #1E3A8A, #3B82F6, #1E40AF, #60A5FA);
  background-size: 400% 400%;
  animation: gradientMove 15s ease infinite;
}

.app.bg-snow {
  background: linear-gradient(-45deg, #60A5FA, #BFDBFE, #93C5FD, #E0F2FE);
  background-size: 400% 400%;
  animation: gradientMove 15s ease infinite;
}

.app.bg-storm {
  background: linear-gradient(-45deg, #1F2937, #4B5563, #111827, #374151);
  background-size: 400% 400%;
  animation: gradientMove 15s ease infinite;
}

.app.bg-fog {
  background: linear-gradient(-45deg, #6B7280, #9CA3AF, #D1D5DB, #4B5563);
  background-size: 400% 400%;
  animation: gradientMove 15s ease infinite;
}

@keyframes gradientMove {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* ============ لایه‌های تزئینی ============ */
.bg-layer {
  position: absolute;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
  z-index: 0;
}

.bubble {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(4px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  animation: floatBubble 20s infinite ease-in-out;
}

.bubble-1 { width: 200px; height: 200px; top: 10%; left: 5%; animation-duration: 22s; }
.bubble-2 { width: 120px; height: 120px; top: 60%; left: 15%; animation-duration: 18s; animation-delay: -5s; }
.bubble-3 { width: 280px; height: 280px; top: 20%; right: 8%; animation-duration: 25s; animation-delay: -10s; }
.bubble-4 { width: 100px; height: 100px; bottom: 15%; right: 20%; animation-duration: 16s; animation-delay: -3s; }
.bubble-5 { width: 160px; height: 160px; bottom: 5%; left: 30%; animation-duration: 20s; animation-delay: -8s; }

@keyframes floatBubble {
  0%, 100% { transform: translate(0, 0) scale(1); }
  25% { transform: translate(30px, -40px) scale(1.05); }
  50% { transform: translate(-20px, -80px) scale(0.95); }
  75% { transform: translate(20px, -40px) scale(1.02); }
}

.glow {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.5;
  animation: glowPulse 8s ease-in-out infinite;
}

.glow-1 { width: 400px; height: 400px; background: rgba(255, 255, 255, 0.3); top: -100px; left: -100px; }
.glow-2 { width: 500px; height: 500px; background: rgba(255, 255, 255, 0.15); bottom: -150px; right: -150px; animation-delay: -3s; }
.glow-3 { width: 300px; height: 300px; background: rgba(255, 255, 255, 0.2); top: 40%; left: 40%; animation-delay: -5s; }

@keyframes glowPulse {
  0%, 100% { opacity: 0.3; transform: scale(1); }
  50% { opacity: 0.6; transform: scale(1.1); }
}

/* ============ کانتینر ============ */
.container {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 700px;
  padding: 2.5rem 2rem;
  border-radius: 28px;
  background: rgba(255, 255, 255, 0.18);
  backdrop-filter: blur(24px) saturate(180%);
  -webkit-backdrop-filter: blur(24px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.35);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15), inset 0 1px 0 rgba(255, 255, 255, 0.5);
  text-align: center;
  animation: fadeInUp 0.6s ease;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(30px); }
  to { opacity: 1; transform: translateY(0); }
}

/* ============ تیتر ============ */
.title {
  font-size: 1.8rem;
  margin-bottom: 1.25rem;
  color: #fff;
  font-weight: 800;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.icon-title {
  font-size: 2rem;
  animation: float 3s ease-in-out infinite;
}

/* ============ Mode Tabs ============ */
.mode-tabs {
  display: flex;
  gap: 0.5rem;
  padding: 0.35rem;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 50px;
  margin-bottom: 1.5rem;
}

.mode-tab {
  flex: 1;
  padding: 0.7rem 1rem;
  background: transparent;
  border: none;
  border-radius: 50px;
  color: rgba(255, 255, 255, 0.7);
  font-family: inherit;
  font-size: 0.88rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.mode-tab:hover {
  color: #fff;
}

.mode-tab.active {
  background: rgba(255, 255, 255, 0.95);
  color: #1a7a5c;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

/* ============ جستجو ============ */
.search-box {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1.5rem;
}

.search-box input {
  flex: 1;
  padding: 0.95rem 1.25rem;
  border: 1px solid rgba(255, 255, 255, 0.4);
  border-radius: 14px;
  font-size: 1rem;
  font-family: inherit;
  background: rgba(255, 255, 255, 0.9);
  transition: all 0.3s;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.search-box input:focus {
  outline: none;
  background: #fff;
  border-color: rgba(255, 255, 255, 0.8);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.search-box button {
  padding: 0.95rem 1.5rem;
  background: rgba(255, 255, 255, 0.95);
  color: #1a7a5c;
  border: none;
  border-radius: 14px;
  font-size: 0.95rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  white-space: nowrap;
}

.search-box button:hover:not(:disabled) {
  background: #fff;
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
}

.search-box button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* ============ تاریخچه ============ */
.history-section {
  margin-bottom: 1.5rem;
  padding: 1rem 1.25rem;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.25);
  border-radius: 16px;
}

.history-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.75rem;
}

.history-title {
  color: #fff;
  font-size: 0.85rem;
  font-weight: 600;
}

.clear-btn {
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.75);
  font-size: 0.75rem;
  font-family: inherit;
  cursor: pointer;
  padding: 0.25rem 0.5rem;
  border-radius: 8px;
  transition: all 0.3s;
}

.clear-btn:hover {
  color: #fff;
  background: rgba(255, 255, 255, 0.15);
}

.history-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.history-item {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.5rem 0.85rem;
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: 50px;
  font-size: 0.82rem;
  font-weight: 600;
  color: #2c3e50;
  cursor: pointer;
  transition: all 0.3s;
  font-family: inherit;
}

.history-item:hover {
  background: #fff;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.history-remove {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.08);
  font-size: 0.7rem;
  transition: all 0.3s;
}

.history-remove:hover {
  background: #ef4444;
  color: #fff;
}

/* ============ Loading ============ */
.loading {
  padding: 2rem;
  color: #fff;
}

.spinner {
  width: 44px;
  height: 44px;
  margin: 0 auto 1rem;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* ============ Error ============ */
.error {
  padding: 1.5rem;
  background: rgba(254, 226, 226, 0.95);
  color: #b91c1c;
  border-radius: 14px;
  font-size: 1rem;
  font-weight: 500;
}

/* ============ Weather Card ============ */
.weather-card {
  padding: 2rem 1.5rem;
  background: rgba(255, 255, 255, 0.25);
  backdrop-filter: blur(10px);
  border-radius: 22px;
  border: 1px solid rgba(255, 255, 255, 0.4);
  animation: fadeIn 0.6s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}

.city-name {
  font-size: 1.75rem;
  margin-bottom: 1rem;
  color: #fff;
  font-weight: 800;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.city-name small {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.85);
  font-weight: 400;
}

.weather-icon {
  width: 130px;
  height: 130px;
  margin: 0 auto 1rem;
  animation: float 3s ease-in-out infinite;
}

.weather-icon svg {
  width: 100%;
  height: 100%;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.temp {
  font-size: 4.5rem;
  font-weight: 900;
  color: #fff;
  line-height: 1;
  margin-bottom: 0.5rem;
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.25);
  letter-spacing: -2px;
}

.description {
  font-size: 1.2rem;
  color: #fff;
  margin-bottom: 1.5rem;
  font-weight: 600;
}

.details {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  margin-top: 1.5rem;
  padding-top: 1.5rem;
  border-top: 1px solid rgba(255, 255, 255, 0.3);
}

.detail-item {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  padding: 0.5rem;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 12px;
}

.detail-item span {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.85);
}

.detail-item strong {
  font-size: 1.05rem;
  color: #fff;
  font-weight: 700;
}

/* ============ Forecast ============ */
.forecast-section {
  margin-top: 1.5rem;
  padding: 1.5rem 1.25rem;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 22px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  animation: fadeIn 0.8s ease;
}

.forecast-title {
  color: #fff;
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 1.25rem;
  text-align: center;
}

.forecast-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 0.75rem;
}

.forecast-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem 0.5rem;
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 16px;
  transition: all 0.3s ease;
}

.forecast-card:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-4px);
}

.forecast-day {
  color: #fff;
  font-size: 0.78rem;
  font-weight: 600;
  white-space: nowrap;
}

.forecast-icon {
  width: 44px;
  height: 44px;
}

.forecast-icon svg {
  width: 100%;
  height: 100%;
}

.forecast-temps {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.temp-max {
  color: #fff;
  font-size: 1rem;
  font-weight: 800;
}

.temp-min {
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.85rem;
  font-weight: 600;
}

/* ============ Outfit ============ */
.outfit-section {
  margin-top: 1.5rem;
  padding: 1.75rem 1.5rem;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 22px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  animation: fadeIn 1s ease;
}

.outfit-title {
  color: #fff;
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 1.25rem;
  text-align: center;
}

.outfit-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(90px, 1fr));
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.outfit-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem 0.5rem;
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 16px;
  transition: all 0.3s ease;
}

.outfit-item:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-4px);
}

.outfit-icon-wrapper {
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
}

.outfit-label {
  color: #fff;
  font-size: 0.75rem;
  font-weight: 600;
  text-align: center;
  line-height: 1.3;
}

.outfit-tips {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding-top: 1rem;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
}

.outfit-tip {
  color: #fff;
  font-size: 0.85rem;
  font-weight: 500;
  padding: 0.5rem 0.85rem;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 10px;
  text-align: center;
}

/* ============ Compare Mode ============ */
.compare-inputs {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 0.75rem;
  align-items: end;
  margin-bottom: 1rem;
}

.compare-input-group {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  text-align: right;
}

.compare-input-group label {
  color: rgba(255, 255, 255, 0.85);
  font-size: 0.78rem;
  font-weight: 600;
  padding-right: 0.25rem;
}

.compare-input-group input {
  padding: 0.85rem 1rem;
  border: 1px solid rgba(255, 255, 255, 0.4);
  border-radius: 14px;
  font-size: 0.95rem;
  font-family: inherit;
  background: rgba(255, 255, 255, 0.9);
  transition: all 0.3s;
}

.compare-input-group input:focus {
  outline: none;
  background: #fff;
  border-color: rgba(255, 255, 255, 0.8);
  transform: translateY(-2px);
}

.swap-btn {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(255, 255, 255, 0.5);
  color: #1a7a5c;
  font-size: 1.3rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  margin-bottom: 2px;
}

.swap-btn:hover {
  background: #fff;
  transform: rotate(180deg);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.compare-btn {
  width: 100%;
  padding: 1rem 1.5rem;
  background: rgba(255, 255, 255, 0.95);
  color: #1a7a5c;
  border: none;
  border-radius: 14px;
  font-size: 1rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  margin-bottom: 1.5rem;
}

.compare-btn:hover:not(:disabled) {
  background: #fff;
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
}

.compare-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* نتایج مقایسه */
.compare-results {
  animation: fadeIn 0.6s ease;
}

.compare-grid {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 1rem;
  align-items: center;
  margin-bottom: 1.5rem;
}

.compare-card {
  padding: 1.5rem 1rem;
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.35);
  border-radius: 20px;
  text-align: center;
  transition: all 0.3s ease;
}

.compare-card:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-4px);
}

.compare-city {
  color: #fff;
  font-size: 1.1rem;
  font-weight: 800;
  margin-bottom: 0.75rem;
  text-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
}

.compare-icon {
  width: 70px;
  height: 70px;
  margin: 0 auto 0.75rem;
}

.compare-icon svg {
  width: 100%;
  height: 100%;
}

.compare-temp {
  color: #fff;
  font-size: 2.5rem;
  font-weight: 900;
  line-height: 1;
  margin-bottom: 0.5rem;
  text-shadow: 0 3px 12px rgba(0, 0, 0, 0.2);
}

.compare-desc {
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.85rem;
  font-weight: 600;
  margin-bottom: 0.75rem;
}

.compare-details {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  padding-top: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  font-size: 0.78rem;
  color: rgba(255, 255, 255, 0.9);
}

.compare-details div {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.35rem;
}

.vs-badge {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.95);
  color: #1a7a5c;
  font-size: 0.85rem;
  font-weight: 900;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
  letter-spacing: 1px;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.08); }
}

/* خلاصه مقایسه */
.compare-summary {
  padding: 1.5rem;
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  text-align: right;
}

.summary-title {
  color: #fff;
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 1rem;
  text-align: center;
}

.summary-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.15);
}

.summary-item:last-child {
  border-bottom: none;
}

.summary-label {
  color: rgba(255, 255, 255, 0.85);
  font-size: 0.88rem;
}

.summary-value {
  color: #fff;
  font-size: 1rem;
  font-weight: 700;
}

/* ============ Empty ============ */
.empty {
  padding: 2rem 1rem;
  color: #fff;
}

.empty-icon {
  font-size: 3.5rem;
  margin-bottom: 1rem;
  animation: float 4s ease-in-out infinite;
}

.empty p {
  font-size: 1.1rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
}

.empty-hint {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.75);
}

/* ============ موبایل ============ */
@media (max-width: 600px) {
  .container {
    padding: 2rem 1.25rem;
    border-radius: 22px;
  }

  .title {
    font-size: 1.5rem;
  }

  .icon-title {
    font-size: 1.7rem;
  }

  .search-box {
    flex-direction: column;
  }

  .temp {
    font-size: 3.5rem;
  }

  .weather-icon {
    width: 100px;
    height: 100px;
  }

  .details {
    grid-template-columns: 1fr;
  }

  .forecast-grid {
    display: flex;
    overflow-x: auto;
    scroll-snap-type: x mandatory;
    padding-bottom: 0.5rem;
    gap: 0.75rem;
  }

  .forecast-card {
    min-width: 90px;
    flex-shrink: 0;
    scroll-snap-align: start;
  }

  .outfit-grid {
    grid-template-columns: repeat(3, 1fr);
  }

  .bubble-1, .bubble-3 {
    width: 150px;
    height: 150px;
  }

  /* Compare در موبایل */
  .compare-inputs {
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .swap-btn {
    margin: 0 auto;
    transform: rotate(90deg);
  }

  .swap-btn:hover {
    transform: rotate(270deg);
  }

  .compare-grid {
    grid-template-columns: 1fr;
    gap: 0.75rem;
  }

  .vs-badge {
    margin: 0 auto;
  }

  .compare-card {
    padding: 1.25rem 1rem;
  }

  .compare-temp {
    font-size: 2rem;
  }
}
</style>