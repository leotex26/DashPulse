<template>
  <div class="weather-card">
    <div v-if="loading" class="loading">Chargement...</div>
    <div v-else-if="error" class="error">Météo indisponible</div>

    <div v-else class="profile">
      <!-- En-tête : Icône + Nom de ville -->
      <div class="weather-icon">{{ weatherIcon }}</div>
      <h3 class="city">{{ city }}</h3>

      <!-- Température principale (remplace la bio) -->
      <p class="temp-display">{{ Math.round(temperature) }}°C</p>

      <!-- Section statistiques (identique à la carte GitHub) -->
      <div class="stats">
        <div class="stat">
          <span class="count">{{ windspeed }}</span>
          <span class="label">km/h Vent</span>
        </div>
        <div class="stat">
          <span class="count">{{ humidity }}%</span>
          <span class="label">Humidité</span>
        </div>
      </div>

      <!-- État du ciel sous forme de petit badge -->
      <div class="weather-status">
        {{ weatherDescription }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const city = ref('Rennes')
const lat = 48.1173
const lon = -1.6778

const temperature = ref(null)
const windspeed = ref(null)
const humidity = ref(null)
const weatherCode = ref(null)
const loading = ref(true)
const error = ref(false)

// Conversion du code météo WMO en icône
const weatherIcon = computed(() => {
  const code = weatherCode.value
  if (code === 0) return '☀️'
  if ([1, 2, 3].includes(code)) return '⛅'
  if ([45, 48].includes(code)) return '🌫️'
  if ([51, 53, 55, 61, 63, 65].includes(code)) return '🌧️'
  if ([71, 73, 75].includes(code)) return '❄️'
  if ([95, 96, 99].includes(code)) return '⛈️'
  return '🌡️'
})

// Description texte associée au code
const weatherDescription = computed(() => {
  const code = weatherCode.value
  if (code === 0) return 'Ciel dégagé'
  if ([1, 2, 3].includes(code)) return 'Partiellement nuageux'
  if ([45, 48].includes(code)) return 'Brouillard'
  if ([51, 53, 55, 61, 63, 65].includes(code)) return 'Pluie'
  if ([71, 73, 75].includes(code)) return 'Neige'
  if ([95, 96, 99].includes(code)) return 'Orage'
  return 'Météo'
})

const fetchWeather = async () => {
  try {
    const res = await fetch(
      `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,relative_humidity_2m,weather_code,wind_speed_10m`
    )
    if (!res.ok) throw new Error()
    const data = await res.json()

    temperature.value = data.current.temperature_2m
    humidity.value = data.current.relative_humidity_2m
    windspeed.value = Math.round(data.current.wind_speed_10m)
    weatherCode.value = data.current.weather_code
  } catch (err) {
    error.value = true
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchWeather()
})
</script>

<style scoped>
.weather-card {
  background: #ffffff;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.06);
  width: 280px; /* Même largeur compacte que GitHub */
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  border: 1px solid #e1e4e8;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  animation: fadeIn 0.5s ease-in-out;
}

.weather-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.12);
}

.profile {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.weather-icon {
  font-size: 2.8rem;
  line-height: 1;
  margin-bottom: 8px;
}

.city {
  margin: 0;
  color: #24292e;
  font-size: 1.2rem;
  font-weight: 700;
}

.temp-display {
  font-size: 1.8rem;
  font-weight: 800;
  color: #24292e;
  margin: 4px 0 8px 0;
}

.stats {
  display: flex;
  justify-content: space-around;
  width: 100%;
  margin: 12px 0 16px 0;
  border-top: 1px solid #f0f0f0;
  border-bottom: 1px solid #f0f0f0;
  padding: 12px 0;
}

.stat {
  display: flex;
  flex-direction: column;
}

.count {
  font-weight: 800;
  font-size: 1.2em;
  color: #24292e;
}

.label {
  font-size: 0.75em;
  color: #57606a;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.weather-status {
  display: inline-block;
  background: #f6f8fa;
  color: #57606a;
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 0.85em;
  font-weight: 600;
  width: 100%;
  box-sizing: border-box;
}

.loading, .error {
  text-align: center;
  color: #57606a;
  font-size: 0.9em;
  padding: 20px 0;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>























