<template>
  <article class="weather-card" aria-label="Météo actuelle">
    <!-- Chargement -->
    <div v-if="loading" class="state loading" aria-live="polite">
      <div class="spinner" aria-hidden="true"></div>
      <span>Chargement de la météo...</span>
    </div>

    <!-- Erreur -->
    <div v-else-if="error" class="state error" aria-live="assertive">
      <span class="error-icon" aria-hidden="true">⚠️</span>
      <p>Impossible de récupérer la météo.</p>

      <button class="retry-button" type="button" @click="fetchWeather">
        Réessayer
      </button>
    </div>

    <!-- Météo -->
    <div v-else class="weather-content">

      <!-- Header -->
      <header class="weather-header">
        <div
          class="weather-icon"
          :aria-label="weatherDescription"
          role="img"
        >
          {{ weatherIcon }}
        </div>

        <div class="location">
          <span class="location-label">Météo actuelle</span>
          <h2 class="city">{{ city }}</h2>
        </div>
      </header>

      <!-- Température -->
      <div class="temperature">
        <span class="temperature-value">
          {{ Math.round(temperature ?? 0) }}
        </span>

        <span class="temperature-unit">°C</span>
      </div>

      <!-- Description -->
      <p class="weather-status">
        {{ weatherDescription }}
      </p>

      <!-- Statistiques -->
      <div class="stats">

        <div class="stat">
          <span class="stat-icon" aria-hidden="true">💨</span>

          <div>
            <span class="count">
              {{ windspeed }} km/h
            </span>

            <span class="label">
              Vent
            </span>
          </div>
        </div>

        <div class="stat">
          <span class="stat-icon" aria-hidden="true">💧</span>

          <div>
            <span class="count">
              {{ humidity }}%
            </span>

            <span class="label">
              Humidité
            </span>
          </div>
        </div>

      </div>

    </div>
  </article>
</template>

<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

/* -------------------------------------------------------------------------- */
/*                                  Props                                     */
/* -------------------------------------------------------------------------- */

interface Props {
  city?: string
  latitude?: number
  longitude?: number
}

const props = withDefaults(defineProps<Props>(), {
  city: 'Rennes',
  latitude: 48.1173,
  longitude: -1.6778,
})

/* -------------------------------------------------------------------------- */
/*                              Types météo                                   */
/* -------------------------------------------------------------------------- */

interface WeatherData {
  temperature: number
  humidity: number
  windspeed: number
  weatherCode: number
}

interface WeatherDescription {
  icon: string
  label: string
}

/* -------------------------------------------------------------------------- */
/*                             Weather mapping                                */
/* -------------------------------------------------------------------------- */

/**
 * Codes météo WMO utilisés par Open-Meteo.
 *
 * https://open-meteo.com/en/docs
 */
const WEATHER_CODES: Record<number, WeatherDescription> = {
  0: {
    icon: '☀️',
    label: 'Ciel dégagé',
  },

  1: {
    icon: '🌤️',
    label: 'Principalement dégagé',
  },

  2: {
    icon: '⛅',
    label: 'Partiellement nuageux',
  },

  3: {
    icon: '☁️',
    label: 'Couvert',
  },

  45: {
    icon: '🌫️',
    label: 'Brouillard',
  },

  48: {
    icon: '🌫️',
    label: 'Brouillard givrant',
  },

  51: {
    icon: '🌦️',
    label: 'Bruine légère',
  },

  53: {
    icon: '🌦️',
    label: 'Bruine modérée',
  },

  55: {
    icon: '🌧️',
    label: 'Bruine forte',
  },

  61: {
    icon: '🌧️',
    label: 'Pluie légère',
  },

  63: {
    icon: '🌧️',
    label: 'Pluie modérée',
  },

  65: {
    icon: '🌧️',
    label: 'Forte pluie',
  },

  71: {
    icon: '🌨️',
    label: 'Neige légère',
  },

  73: {
    icon: '❄️',
    label: 'Neige modérée',
  },

  75: {
    icon: '❄️',
    label: 'Forte neige',
  },

  95: {
    icon: '⛈️',
    label: 'Orage',
  },

  96: {
    icon: '⛈️',
    label: 'Orage avec grêle légère',
  },

  99: {
    icon: '⛈️',
    label: 'Orage avec forte grêle',
  },
}

/* -------------------------------------------------------------------------- */
/*                                  State                                     */
/* -------------------------------------------------------------------------- */

const temperature = ref<number | null>(null)
const windspeed = ref<number | null>(null)
const humidity = ref<number | null>(null)
const weatherCode = ref<number | null>(null)

const loading = ref(true)
const error = ref(false)

let controller: AbortController | null = null

/* -------------------------------------------------------------------------- */
/*                            Weather computed                                */
/* -------------------------------------------------------------------------- */

const weather = computed<WeatherDescription>(() => {
  if (weatherCode.value === null) {
    return {
      icon: '🌡️',
      label: 'Météo inconnue',
    }
  }

  return (
    WEATHER_CODES[weatherCode.value] ?? {
      icon: '🌡️',
      label: 'Conditions météorologiques inconnues',
    }
  )
})

const weatherIcon = computed(() => weather.value.icon)

const weatherDescription = computed(() => weather.value.label)

const city = computed(() => props.city)

/* -------------------------------------------------------------------------- */
/*                              Fetch weather                                 */
/* -------------------------------------------------------------------------- */

const fetchWeather = async () => {
  loading.value = true
  error.value = false

  controller?.abort()
  controller = new AbortController()

  try {
    const params = new URLSearchParams({
      latitude: String(props.latitude),
      longitude: String(props.longitude),
      current:
        'temperature_2m,relative_humidity_2m,weather_code,wind_speed_10m',
      timezone: 'auto',
    })

    const response = await fetch(
      `https://api.open-meteo.com/v1/forecast?${params.toString()}`,
      {
        signal: controller.signal,
      }
    )

    if (!response.ok) {
      throw new Error(
        `Erreur HTTP ${response.status}`
      )
    }

    const data = await response.json()

    if (!data.current) {
      throw new Error('Données météo indisponibles')
    }

    const weatherData: WeatherData = {
      temperature: data.current.temperature_2m,
      humidity: data.current.relative_humidity_2m,
      windspeed: Math.round(data.current.wind_speed_10m),
      weatherCode: data.current.weather_code,
    }

    temperature.value = weatherData.temperature
    humidity.value = weatherData.humidity
    windspeed.value = weatherData.windspeed
    weatherCode.value = weatherData.weatherCode

  } catch (err) {
    /**
     * Une AbortError est normale lorsque le composant
     * est démonté ou qu'une nouvelle requête est lancée.
     */
    if (err instanceof DOMException && err.name === 'AbortError') {
      return
    }

    console.error('Erreur lors de la récupération météo:', err)

    error.value = true

  } finally {
    loading.value = false
  }
}

/* -------------------------------------------------------------------------- */
/*                                Lifecycle                                   */
/* -------------------------------------------------------------------------- */

onMounted(fetchWeather)

onBeforeUnmount(() => {
  controller?.abort()
})
</script>

<style scoped>
/* -------------------------------------------------------------------------- */
/*                                  Card                                      */
/* -------------------------------------------------------------------------- */

.weather-card {
  width: min(100%, 320px);
  box-sizing: border-box;

  padding: 24px;

  background:
    linear-gradient(
      145deg,
      #ffffff 0%,
      #f8fafc 100%
    );

  border: 1px solid #e5e7eb;
  border-radius: 20px;

  box-shadow:
    0 4px 6px rgba(0, 0, 0, 0.04),
    0 12px 30px rgba(0, 0, 0, 0.06);

  font-family:
    -apple-system,
    BlinkMacSystemFont,
    "Segoe UI",
    Roboto,
    Helvetica,
    Arial,
    sans-serif;

  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease;
}

.weather-card:hover {
  transform: translateY(-4px);

  box-shadow:
    0 8px 12px rgba(0, 0, 0, 0.05),
    0 20px 40px rgba(0, 0, 0, 0.1);
}

/* -------------------------------------------------------------------------- */
/*                                  Header                                    */
/* -------------------------------------------------------------------------- */

.weather-header {
  display: flex;
  align-items: center;
  gap: 14px;
}

.weather-icon {
  display: flex;
  align-items: center;
  justify-content: center;

  width: 64px;
  height: 64px;

  border-radius: 18px;

  background: #f0fdf4;

  font-size: 2.4rem;

  flex-shrink: 0;
}

.location {
  min-width: 0;
}

.location-label {
  display: block;

  margin-bottom: 3px;

  color: #94a3b8;

  font-size: 0.7rem;
  font-weight: 700;

  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.city {
  margin: 0;

  color: #0f172a;

  font-size: 1.25rem;
  font-weight: 800;

  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* -------------------------------------------------------------------------- */
/*                              Temperature                                   */
/* -------------------------------------------------------------------------- */

.temperature {
  display: flex;
  align-items: flex-start;

  margin-top: 24px;
}

.temperature-value {
  color: #0f172a;

  font-size: 4rem;
  font-weight: 800;

  line-height: 0.95;

  letter-spacing: -0.06em;
}

.temperature-unit {
  margin-top: 4px;

  color: #64748b;

  font-size: 1.3rem;
  font-weight: 600;
}

/* -------------------------------------------------------------------------- */
/*                               Description                                  */
/* -------------------------------------------------------------------------- */

.weather-status {
  display: inline-flex;

  margin: 14px 0 22px;
  padding: 7px 12px;

  background: #ecfdf5;
  color: #047857;

  border-radius: 999px;

  font-size: 0.8rem;
  font-weight: 700;
}

/* -------------------------------------------------------------------------- */
/*                                  Stats                                     */
/* -------------------------------------------------------------------------- */

.stats {
  display: grid;
  grid-template-columns: 1fr 1fr;

  border-top: 1px solid #e5e7eb;

  padding-top: 18px;

  gap: 16px;
}

.stat {
  display: flex;
  align-items: center;
  gap: 10px;
}

.stat-icon {
  display: flex;
  align-items: center;
  justify-content: center;

  width: 36px;
  height: 36px;

  background: #f8fafc;

  border-radius: 10px;

  font-size: 1rem;
}

.stat > div {
  display: flex;
  flex-direction: column;
}

.count {
  color: #0f172a;

  font-size: 0.9rem;
  font-weight: 800;
}

.label {
  margin-top: 2px;

  color: #94a3b8;

  font-size: 0.65rem;
  font-weight: 700;

  text-transform: uppercase;
  letter-spacing: 0.05em;
}

/* -------------------------------------------------------------------------- */
/*                                  States                                    */
/* -------------------------------------------------------------------------- */

.state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  min-height: 220px;

  text-align: center;

  color: #64748b;

  font-size: 0.9rem;
  font-weight: 600;
}

.error {
  color: #b91c1c;
}

.error-icon {
  margin-bottom: 8px;

  font-size: 2rem;
}

.error p {
  margin: 0 0 14px;
}

/* -------------------------------------------------------------------------- */
/*                                  Spinner                                   */
/* -------------------------------------------------------------------------- */

.spinner {
  width: 28px;
  height: 28px;

  margin-bottom: 14px;

  border: 3px solid #e2e8f0;
  border-top-color: #10b981;

  border-radius: 50%;

  animation: spin 0.8s linear infinite;
}

/* -------------------------------------------------------------------------- */
/*                                  Button                                    */
/* -------------------------------------------------------------------------- */

.retry-button {
  padding: 8px 14px;

  border: 0;
  border-radius: 10px;

  background: #10b981;
  color: white;

  font-size: 0.8rem;
  font-weight: 700;

  cursor: pointer;

  transition:
    background 0.2s ease,
    transform 0.2s ease;
}

.retry-button:hover {
  background: #059669;
  transform: translateY(-1px);
}

.retry-button:focus-visible {
  outline: 3px solid rgba(16, 185, 129, 0.3);
  outline-offset: 2px;
}

/* -------------------------------------------------------------------------- */
/*                                Animations                                  */
/* -------------------------------------------------------------------------- */

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* -------------------------------------------------------------------------- */
/*                              Responsive                                    */
/* -------------------------------------------------------------------------- */

@media (max-width: 360px) {
  .weather-card {
    padding: 20px;
  }

  .temperature-value {
    font-size: 3.4rem;
  }

  .stats {
    grid-template-columns: 1fr;
  }
}

@media (prefers-reduced-motion: reduce) {
  .weather-card,
  .retry-button {
    transition: none;
  }

  .spinner {
    animation: none;
  }
}
</style>