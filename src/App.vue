<script setup>
import { ref, onMounted } from 'vue'
import { supabase } from './lib/supaBaseClient'
import '@/assets/style.css'
import LoginModal from './components/LoginModal.vue'
import AdminModal from './components/AdminModal.vue'

const applications = ref([])
const showLogin = ref(false)
const showAdmin = ref(false)
const session = ref(null)

async function getApplications() {
  const { data } = await supabase.from('applications').select()
  applications.value = data
}

function handleLoginSuccess(s) {
  session.value = s
}

function logout() {
  supabase.auth.signOut()
  session.value = null
}

onMounted(async () => {
  getApplications()
  const { data } = await supabase.auth.getSession()
  session.value = data.session
})
</script>

<template>
  <div class="main-container">
    <header>
      <h1>Portail des applications TICE</h1>
      <div class="auth-buttons">
        <button v-if="!session" @click="showLogin = true">Se connecter</button>
        <template v-else>
          <button @click="showAdmin = true">Administration</button>
          <button @click="logout">Se déconnecter</button>
        </template>
      </div>
    </header>

    <h2>Plateformes utilisateurs</h2>
    <div id="services-users">
      <a v-for="app in applications.filter(app => app.useradmin).sort((a, b) => a.title.localeCompare(b.title))"
         :key="app.id"
         :href="app.url"
         class="service">
        <img :src="app.img">
        <span>{{ app.title }}</span>
      </a>
    </div>

    <h2>Plateformes administration</h2>
    <div id="services-admin">
      <a v-for="app in applications.filter(app => !app.useradmin).sort((a, b) => a.title.localeCompare(b.title))"
         :key="app.id"
         :href="app.url"
         class="service">
        <img :src="app.img">
        <span>{{ app.title }}</span>
      </a>
    </div>
  </div>

  <!-- Modale login -->
  <LoginModal 
    v-if="showLogin"
    @close="showLogin = false"
    @loginSuccess="handleLoginSuccess"
  />

  <!-- Modale admin -->
<AdminModal 
  v-if="showAdmin"
  @close="showAdmin = false"
  @updated="getApplications"
/>
</template>
