<script setup>
/**
 * ARCHIVO: App.vue
 * DESCRIPCIÓN: Punto de entrada principal. Aquí controlamos el "Estado Global" 
 * de la sesión para saber si mostramos la puerta (Login) o el panel de control.
 */
import { ref } from 'vue'
import Login from './components/Login.vue'
import AdminPanel from './components/AdminPanel.vue'

// "usuarioActivo" es la variable que me guarda la session si es null, no me enseña nada.
const usuarioActivo = ref(null)

/**
 * Recibe el testigo del componente Login.
 * datos del servidor me da el usuario que ha hecho login correctamente
 */
const iniciarSesion = (datosDelServidor) => {
  console.log("Sesión iniciada para:", datosDelServidor.login)
  usuarioActivo.value = datosDelServidor
}

/**
 * Borra el rastro del usuario. Al ser reactivo (ref), 
 * Vue detecta el cambio y nos manda directos al Login por el v-if.
 */
const cerrarSesion = () => {
  if (confirm("¿Seguro que quieres salir?")) {
    usuarioActivo.value = null
  }
}
</script>

<template>
  <main class="main-wrapper">
    
    <Login 
      v-if="!usuarioActivo" 
      @login-exitoso="iniciarSesion" 
    />

    <AdminPanel 
      v-else 
      :usuario="usuarioActivo" 
      @logout="cerrarSesion"
    />

  </main>
</template>

<style>
:root {
  --primary: #2c3e50;
  --bg-dark: #1e1e1e; 
  --text-light: #f5f5f5;
}

body {
  margin: 0;
  padding: 0;
  background-color: var(--bg-dark);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: var(--text-light);
}

.main-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 1rem;
  box-sizing: border-box;
}

.main-wrapper > * {
  transition: all 0.3s ease;
}
</style>