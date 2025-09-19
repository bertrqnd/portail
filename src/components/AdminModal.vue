<script setup>
import { ref, onMounted } from 'vue'
import { supabase } from '../lib/supaBaseClient'

const applications = ref([])
const title = ref('')
const url = ref('')
const useradmin = ref(true)
const file = ref(null) // fichier logo choisi
const editingId = ref(null) // null = ajout, sinon édition
const img = ref('') // URL actuelle du logo (si édition)
const emit = defineEmits(['close', 'updated'])
const user = ref(null)
const isAdmin = ref(false)


onMounted(async () => {
  const { data, error } = await supabase.auth.getUser()
  if (error || !data?.user) {
    console.error('Utilisateur non connecté')
    return
  }

  user.value = data.user
  isAdmin.value = data.user.user_metadata?.is_admin === true

  await getApplications()
})



async function getApplications() {
  const { data } = await supabase.from('applications').select()
  applications.value = data
  emit('updated')
}

function onFileChange(e) {
  file.value = e.target.files[0]
}

// Supprime un logo du bucket à partir de son URL publique
async function deleteLogoFromBucket(publicUrl) {
  if (!publicUrl) return
  const urlParts = publicUrl.split('/src/')
  if (urlParts.length > 1) {
    const filePath = urlParts[1] // ex: "logos/123456.png"
    await supabase.storage.from('src').remove([filePath])
  }
}

// Upload du fichier dans Supabase Storage
async function uploadLogo() {
  if (!file.value) return img.value // si aucun fichier choisi → garder l’URL existante

  // Si on est en édition avec un ancien logo → suppression
  if (editingId.value && img.value) {
    await deleteLogoFromBucket(img.value)
  }

  const fileExt = file.value.name.split('.').pop()
  const fileName = `${Date.now()}.${fileExt}`
  const filePath = `logos/${fileName}`

  let { error: uploadError } = await supabase.storage
    .from('src')
    .upload(filePath, file.value, { upsert: true })

  if (uploadError) {
    console.error('Erreur upload:', uploadError.message)
    return img.value
  }

  // Récupérer URL publique
  const { data } = supabase.storage.from('src').getPublicUrl(filePath)
  return data.publicUrl
}

async function saveApplication() {
  const logoUrl = await uploadLogo()

  if (editingId.value) {
    // Modification
    await supabase.from('applications')
      .update({
        title: title.value,
        url: url.value,
        img: logoUrl,
        useradmin: useradmin.value
      })
      .eq('id', editingId.value)
  } else {
    // Ajout
    await supabase.from('applications').insert({
      title: title.value,
      url: url.value,
      img: logoUrl,
      useradmin: useradmin.value,
      user_id: user.id
    })
  }

  await getApplications()
  resetForm()
}

function editApplication(app) {
  editingId.value = app.id
  title.value = app.title
  url.value = app.url
  img.value = app.img
  useradmin.value = app.useradmin
}

// Suppression appli + logo
async function deleteApplication(app) {
  if (app.img) {
    await deleteLogoFromBucket(app.img)
  }

  await supabase.from('applications').delete().eq('id', app.id)
  await getApplications()
}

function resetForm() {
  editingId.value = null
  title.value = ''
  url.value = ''
  useradmin.value = true
  img.value = ''
  file.value = null
}

onMounted(() => {
  getApplications()
})
</script>

<template>
  <div class="modal-overlay" @click.self="emit('close')">
    <div class="modal">
      <h2>Gestion des applications</h2>

      <!-- Formulaire ajout / édition -->
      <form @submit.prevent="saveApplication">
        <input v-model="title" placeholder="Titre" required />
        <input v-model="url" placeholder="URL" required />
        <label>
          <input type="checkbox" v-model="useradmin" />
          Application utilisateur ?
        </label>

        <div>
          <label>Logo :</label>
          <input type="file" @change="onFileChange" />
          <div v-if="img">
            <small>Logo actuel :</small><br>
            <img :src="img" alt="logo" width="60" />
          </div>
        </div>

        <button type="submit">{{ editingId ? 'Mettre à jour' : 'Ajouter' }}</button>
        <button type="button" @click="resetForm" v-if="editingId">Annuler édition</button>
      </form>

      <!-- Liste des applis -->
      <ul>
        <li v-for="app in applications" :key="app.id">
          <img :src="app.img" width="20" />
          {{ app.title }}
          <button @click="editApplication(app)">Modifier</button>
          <button @click="deleteApplication(app)">Supprimer</button>
        </li>
      </ul>

      <button @click="emit('close')">Fermer</button>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.6);
  display: flex;
  align-items: center;
  justify-content: center;
}
.modal {
  background: white;
  padding: 2rem;
  border-radius: 10px;
  min-width: 450px;
  max-height: 90vh;
  overflow-y: auto;
}
form {
  margin-bottom: 1.5rem;
}
ul {
  list-style: none;
  padding: 0;
}

li {
  display: flex;
  margin: 0.25rem 0;
}
img {
  border-radius: 6px;
}
</style>
