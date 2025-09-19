<script setup>
import { ref } from 'vue'
import { supabase } from '../lib/supaBaseClient'

const email = ref('')
const password = ref('')
const errorMsg = ref('')
const emit = defineEmits(['close', 'loginSuccess'])

async function login() {
  const { data, error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value
  })

  if (error) {
    errorMsg.value = error.message
  } else {
    emit('loginSuccess', data.session)
    emit('close')
  }
}
</script>

<template>
  <div class="modal-overlay" @click.self="emit('close')">
    <div class="modal">
      <h2>Connexion administrateur</h2>
      <form @submit.prevent="login">
        <input type="email" v-model="email" placeholder="Email" required />
        <input type="password" v-model="password" placeholder="Mot de passe" required />
        <button type="submit">Se connecter</button>
        <button type="button" @click="emit('close')">Annuler</button>
      </form>
      <p v-if="errorMsg" class="error">{{ errorMsg }}</p>
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
  min-width: 300px;
}
.error {
  color: red;
  margin-top: 0.5rem;
}
</style>
