<script setup>
import { ref } from 'vue'
import Login from './components/Login.vue'
import AdminPanel from './components/AdminPanel.vue'

// Esta es nuestra única fuente de verdad sobre quién está en la app
const usuarioGlobal = ref(null)

// Cuando el Login.vue nos avisa que el usuario es correcto
const manejarLogin = (datosUsuario) => {
  usuarioGlobal.value = datosUsuario
}

// Función para limpiar el rastro y volver al login
const cerrarSesion = () => {
  usuarioGlobal.value = null
}
</script>

<template>
  <div class="app-container">
    
    <Login 
      v-if="!usuarioGlobal" 
      @login-exitoso="manejarLogin" 
    />

    <AdminPanel 
      v-else 
      :usuario="usuarioGlobal" 
      @logout="cerrarSesion"
    />

  </div>
</template>

<style>
/* Aquí puedes mantener el estilo que te pasé anteriormente */
:root {
  --primary-color: #2c3e50;
  --bg-color: #333;
}

body {
  margin: 0;
  background-color: var(--bg-color);
  font-family: sans-serif;
}

.app-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
}
</style>