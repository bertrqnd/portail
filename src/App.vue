<script setup>

import { ref, onMounted } from 'vue'
import { supabase } from './lib/supaBaseClient'
import '@/assets/style.css'


const applications = ref([])

async function getApplications() {
  const { data } = await supabase.from('applications').select()
  applications.value = data
}

onMounted(() => {
  getApplications()
})

</script>

<template>

  <div class="main-container">

      <Auth></Auth>

    <h1>Portail des applications TICE</h1>

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

</template>