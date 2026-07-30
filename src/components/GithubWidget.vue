<template>
  <div class="github-card">
    <div v-if="loading" class="loading">Chargement des données...</div>
    
    <div v-else-if="error" class="error">
      Impossible de charger le profil.
    </div>

    <div v-else-if="user" class="profile">
      <img :src="user.avatar_url" :alt="user.login" class="avatar" />
      <div class="info">
        <h3>{{ user.name || user.login }}</h3>
        <p class="bio">{{ user.bio || 'Développeur Fullstack' }}</p>
        
        <div class="stats">
          <div class="stat">
            <span class="count">{{ user.public_repos }}</span>
            <span class="label">Repos</span>
          </div>
          <div class="stat">
            <span class="count">{{ user.followers }}</span>
            <span class="label">Followers</span>
          </div>
        </div>

        <a :href="user.html_url" target="_blank" class="btn">Voir sur GitHub</a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const username = 'leotex26' 

const user = ref(null)
const loading = ref(true)
const error = ref(false)

const fetchGithubData = async () => {
  try {
    const response = await fetch(`https://api.github.com/users/${username}`)
    if (!response.ok) throw new Error('Utilisateur non trouvé')
    user.value = await response.json()
  } catch (err) {
    error.value = true
  } finally {
    loading.value = false
  }
}





onMounted(() => {
  fetchGithubData()
})
</script>

<style scoped>
.github-card {
  background: #ffffff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  max-width: 300px;
  font-family: sans-serif;
  border: 1px solid #e1e4e8;
}

.profile {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.avatar {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  margin-bottom: 12px;
}

.bio {
  font-size: 0.9em;
  color: #586069;
  margin: 8px 0;
}

.stats {
  display: flex;
  justify-content: space-around;
  width: 100%;
  margin: 16px 0;
  border-top: 1px solid #eee;
  border-bottom: 1px solid #eee;
  padding: 8px 0;
}

.stat {
  display: flex;
  flex-direction: column;
}

.count {
  font-weight: bold;
  font-size: 1.2em;
}

.label {
  font-size: 0.8em;
  color: #586069;
}

.btn {
  display: inline-block;
  background-color: #2ea44f;
  color: white;
  padding: 8px 16px;
  border-radius: 6px;
  text-decoration: none;
  font-weight: bold;
  font-size: 0.9em;
}

.btn:hover {
  background-color: #2c974b;
}
</style>