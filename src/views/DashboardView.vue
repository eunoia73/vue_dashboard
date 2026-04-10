<template>
  <div class="dashboard">
    <div class="dashboard-header">
      <h1>📊 실시간 대시보드</h1>
      <div class="current-time">
        <div class="date">{{ currentDate }}</div>
        <div class="time">{{ currentTime }}</div>
      </div>
    </div>
  </div>

  <div class="dashboard-grid">
    <!-- 날씨 정보 -->
    <div class="card weather-card">
      <div class="card-header">
        <h2>🌤️ 오늘의 날씨</h2>
        <button @click="fetchWeather" class="refresh-btn" :disabled="loading.weather">
          {{ loading.weather ? '⏳' : '🔄' }}
        </button>
      </div>

      <div v-if="loading.weather" class="loading">
        <div class="spinner">
          <p>날씨 정보를 불러오는 중...</p>
        </div>
      </div>

      <div v-else-if="weather" class="weather-content">
        <div class="weather-main">
          <div class="temperature">{{ weather.temp }}</div>
          <div class="weather-icon">{{ getWeatherIcon(weather.condition) }}</div>
        </div>
        <div class="weather-info">
          <div class="weather-item">
            <span class="label">날씨</span>
            <span class="value">{{ weather.condition }}</span>
          </div>
        </div>
        <div class="weather-item">
          <span class="label">습도</span>
          <span class="value">{{ weather.humidity }}</span>
        </div>
        <div class="weather-item">
          <span class="label">풍속</span>
          <span class="value">{{ weather.windSpeed }}</span>
        </div>
        <div class="weather-item">
          <span class="label">지역</span>
          <span class="value">{{ weather.location }}</span>
        </div>
      </div>

      <div v-else class="error-message">⚠️ 날씨 정보를 불러올 수 없습니다.</div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// 현재 시간
const now = ref(new Date())

// 현재 날짜
const currentDate = computed(() => {
  const year = now.value.getFullYear()
  const month = String(now.value.getMonth() + 1).padStart(2, '0')
  const day = String(now.value.getDate()).padStart(2, '0')
  const weekdays = ['일요일', '월요일', '화요일', '수요일', '목요일', '금요일', '토요일']
  const weekday = weekdays[now.value.getDay()]

  return `${year}년 ${month}월 ${day}일 ${weekday}`
})

// 현재 시간
const currentTime = computed(() => {
  const hours24 = now.value.getHours()
  const minutes = String(now.value.getMinutes()).padStart(2, '0')
  const seconds = String(now.value.getSeconds()).padStart(2, '0')

  // AM/PM 구분
  const period = hours24 >= 12 ? 'PM' : 'AM'

  // 12시간 형식으로 변환(0 -> 12시, 13시 -> 1시)
  const hours12 = hours24 % 12 || 12
  const hoursStr = String(hours12).padStart(2, '0')

  return `${period} ${hoursStr}:${minutes}:${seconds}`
})

// 로딩 상태
const loading = ref({
  weather: false,
  news: false,
})

// 날씨 데이터
const weather = ref(null)

// 뉴스 데이터
// const news = ref([])

// 날씨 아이콘 매핑
function getWeatherIcon(condition) {
  const icons = {
    맑음: '☀️',
    구름많음: '⛅',
    흐림: '☁️',
    비: '🌧️',
    눈: '❄️',
    천둥번개: '⛈️',
  }
  return icons[condition] || '🌤️'
}

// 날씨 정보 가져오기(Open-Meteo API)
async function fetchWeather() {
  loading.value.weather = true

  try {
    // 서울의 위도, 경도
    const latitude = 37.5665
    const longitude = 126.978

    const response = await fetch(
      `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current=temperature_2m,relative_humidity_2m,weather_code,wind_speed_10m&timezone=Asia/Seoul`,
    )

    if (!response.ok) throw new Error('날씨 정보를 가져올 수 없습니다')

    const data = await response.json()

    // 날씨 코드를 한글로 변환
    const weatherCodes = {
      0: '맑음',
      1: '맑음',
      2: '구름많음',
      3: '흐림',
      45: '안개',
      48: '안개',
      51: '이슬비',
      53: '이슬비',
      55: '이슬비',
      61: '비',
      63: '비',
      65: '비',
      71: '눈',
      73: '눈',
      75: '눈',
      95: '천둥번개',
      96: '천둥번개',
      99: '천둥번개',
    }
    weather.value = {
      temp: Math.round(data.current.temperature_2m),
      humidity: data.current.relative_humidity_2m,
      windSpeed: data.current.wind_speed_10m.toFixed(1),
      condition: weatherCodes[data.current.weather_code] || '알 수 없음',
      location: '서울',
    }
  } catch (error) {
    console.error('날씨 정보 로딩 실패:', error)
    weather.value = null
  } finally {
    loading.value.weather = false
  }
}
</script>

<style scoped>
.dashboard {
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.dashboard-header {
  text-align: center;
  padding: 40px 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 16px;
  color: white;
  margin-bottom: 40px;
  box-shadow: 0 8px 30px rgba(102, 126, 234, 0.3);
}

.dashboard-header h1 {
  font-size: 36px;
  margin-bottom: 20px;
}

.current-time {
  display: flex;
  flex-direction: column;
  gap: 10px;
  align-items: center;
}

.date {
  font-size: 24px;
  font-weight: 600;
}

.time {
  font-size: 48px;
  font-weight: bold;
  font-family: 'Courier New', monospace;
  letter-spacing: 2px;
}

.dashboard-grid {
  display: grid;
  gap: 30px;
}

.card {
  background: white;
  border-radius: 16px;
  padding: 30px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  padding-bottom: 15px;
  border-bottom: 2px solid #e2e8f0;
}

.card-header h2 {
  font-size: 24px;
  color: #2d3748;
  margin: 0;
}

.refresh-btn {
  padding: 8px 16px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 18px;
  transition: all 0.3s;
}

.refresh-btn:hover:not(:disabled) {
  background: #5568d3;
  transform: rotate(180deg);
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* 로딩 스피너 */
.loading {
  text-align: center;
  padding: 40px 20px;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 4px solid #e2e8f0;
  border-top: 4px solid #667eea;
  border-radius: 50%;
  margin: 0 auto 20px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.loading p {
  color: #718096;
  font-size: 16px;
}

/* 날씨 카드 */
.weather-content {
  display: grid;
  gap: 30px;
}

.weather-main {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 30px;
  padding: 20px;
  background: linear-gradient(135deg, #667eea20 0%, #764ba220 100%);
  border-radius: 12px;
}

.temperature {
  font-size: 72px;
  font-weight: bold;
  color: #667eea;
}

.weather-icon {
  font-size: 80px;
}

.weather-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

.weather-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  background: #f7fafc;
  border-radius: 10px;
  border-left: 4px solid #667eea;
}

.weather-item .label {
  font-weight: 600;
  color: #4a5568;
}

.weather-item .value {
  color: #667eea;
  font-weight: 600;
  font-size: 18px;
}

/* 뉴스 카드 */
.news-list {
  display: grid;
  gap: 20px;
}

.news-item {
  display: flex;
  gap: 20px;
  padding: 20px;
  background: #f7fafc;
  border-radius: 12px;
  transition: all 0.3s;
  text-decoration: none;
  color: inherit;
  cursor: pointer;
}

.news-item:hover {
  background: #edf2f7;
  transform: translateX(5px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.15);
}

.news-item:hover .news-content h3 {
  color: #667eea;
}

.news-number {
  flex-shrink: 0;
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 18px;
}

.news-content {
  flex: 1;
}

.news-content h3 {
  font-size: 18px;
  color: #2d3748;
  margin-bottom: 8px;
  line-height: 1.4;
}

.news-content p {
  font-size: 14px;
  color: #4a5568;
  line-height: 1.6;
  margin-bottom: 10px;
}

.news-meta {
  display: flex;
  gap: 15px;
  font-size: 13px;
  color: #718096;
}

.news-source {
  font-weight: 600;
}

/* 정보 카드 */
.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 20px;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 25px 20px;
  background: linear-gradient(135deg, #667eea20 0%, #764ba220 100%);
  border-radius: 12px;
  transition: all 0.3s;
}

.info-item:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.2);
}

.info-icon {
  font-size: 40px;
}

.info-label {
  font-size: 14px;
  color: #718096;
  margin-bottom: 5px;
}

.info-value {
  font-size: 24px;
  font-weight: bold;
  color: #2d3748;
}

.error-message {
  text-align: center;
  padding: 40px 20px;
  color: #e53e3e;
  font-size: 16px;
}

/* 반응형 */
@media (max-width: 768px) {
  .dashboard-header h1 {
    font-size: 28px;
  }

  .time {
    font-size: 36px;
  }

  .temperature {
    font-size: 48px;
  }

  .weather-icon {
    font-size: 60px;
  }

  .weather-main {
    flex-direction: column;
  }
}
</style>
