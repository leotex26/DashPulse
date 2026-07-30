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
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.06);
  max-width: 300px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  border: 1px solid #e1e4e8;
  /* Effet de transition fluide pour le survol */
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  animation: fadeIn 0.5s ease-in-out;
}

/* Animation au survol de la carte */
.github-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.12);
}

.profile {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.avatar {
  width: 88px;
  height: 88px;
  border-radius: 50%;
  margin-bottom: 12px;
  border: 3px solid #f0f6fc;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

h3 {
  margin: 0;
  color: #24292e;
  font-size: 1.2rem;
  font-weight: 700;
}

.bio {
  font-size: 0.85em;
  color: #57606a;
  margin: 8px 0 14px 0;
  background: #f6f8fa;
  padding: 4px 12px;
  border-radius: 20px;
}

.stats {
  display: flex;
  justify-content: space-around;
  width: 100%;
  margin: 12px 0 20px 0;
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
  font-size: 1.3em;
  color: #24292e;
}

.label {
  font-size: 0.75em;
  color: #57606a;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.btn {
  display: inline-block;
  background-color: #2da44e;
  color: white;
  padding: 10px 20px;
  border-radius: 8px;
  text-decoration: none;
  font-weight: 600;
  font-size: 0.9em;
  transition: background-color 0.2s ease, transform 0.1s ease;
  width: 100%;
  box-sizing: border-box;
}

.btn:hover {
  background-color: #2c974b;
}

.btn:active {
  transform: scale(0.98);
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

</style>






