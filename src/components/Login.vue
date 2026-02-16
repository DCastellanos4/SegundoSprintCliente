<script setup>
/**
 * COMPONENTE: Login.vue
 * comprobando contra el endpoint de usuarios
 */
import { ref } from 'vue'

// Definimos el evento para avisar a App.vue cuando alguien entre con éxito
const emit = defineEmits(['login-exitoso'])

// Variables reactivas para el formulario
const userField = ref('')
const passField = ref('')
const hasError = ref(false)
const isLoading = ref(false) // para que el botón no se pueda pulsar mil veces

/**
 * Función principal para validar credenciales.
 */
const realizarLogin = async () => {
    hasError.value = false
    isLoading.value = true

    try {
        // consumimos la api
        const respuesta = await fetch('http://100.52.46.68:3000/usuarios')

        if (!respuesta.ok) throw new Error("No se pudo conectar con la API")

        const usuarios = await respuesta.json()

        // se busca si existe el usuario con esa combinación de login/password
        const loginValido = usuarios.find(u =>
            u.login === userField.value && u.password_hash === passField.value
        )

        if (loginValido) {
            console.log("Acceso concedido a:", loginValido.login)
            emit('login-exitoso', loginValido)
        } else {
            hasError.value = true
        }
    } catch (err) {
        console.error("Fallo crítico en el login:", err)
        alert("Error de conexión con el servidor.")
    } finally {
        isLoading.value = false
    }
}
</script>

<template>
    <div class="login-container">
        <div class="login-box">
            <header class="login-header">
                <h2>Escarlatti<span>Gest</span></h2>
                <p>Administración de Centro</p>
            </header>

            <form @submit.prevent="realizarLogin" class="login-form">
                <div class="input-group">
                    <label for="username">Usuario</label>
                    <input id="username" v-model="userField" type="text" placeholder="nombre@centro.es" required>
                </div>

                <div class="input-group">
                    <label for="password">Contraseña</label>
                    <input id="password" v-model="passField" type="password" placeholder="••••••••" required>
                </div>

                <button type="submit" class="btn-login" :disabled="isLoading">
                    <span v-if="!isLoading">Entrar al Sistema</span>
                    <span v-else class="loader">Comprobando...</span>
                </button>
            </form>

            <Transition name="slide-up">
                <div v-if="hasError" class="error-message">
                    <span class="icon">⚠️</span> Error en las credenciales
                </div>
            </Transition>
        </div>
    </div>
</template>

<style scoped>
/* Variables locales para el rediseño */
.login-box {
    --primary: #6366f1;       /* Indigo moderno */
    --primary-hover: #4f46e5;
    --accent: #10b981;        /* Esmeralda */
    --dark-bg: #0f172a;       /* Slate 900 */
    --input-bg: #1e293b;      /* Slate 800 */
    --text-main: #1e293b;
    --text-muted: #64748b;
    --error: #ef4444;
}

.login-container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 80vh;
}

.login-box {
    background: #ffffff;
    padding: 3rem 2.5rem;
    border-radius: 20px;
    box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15);
    width: 100%;
    max-width: 400px;
    text-align: center;
    border: 1px solid #f1f5f9;
}

/* Header con estilo moderno */
.login-header { margin-bottom: 2.5rem; }

.logo-icon {
    background: linear-gradient(135deg, var(--primary), var(--accent));
    width: 50px;
    height: 50px;
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto 1rem;
    color: white;
    font-weight: 800;
    font-size: 1.5rem;
    box-shadow: 0 10px 15px -3px rgba(99, 102, 241, 0.3);
}

.login-header h2 {
    color: var(--dark-bg);
    font-size: 1.8rem;
    letter-spacing: -0.5px;
}

.login-header h2 span { color: var(--primary); }

.login-header p {
    color: var(--text-muted);
    font-size: 0.95rem;
    margin-top: 0.2rem;
}

/* Inputs con estilo "Deep" */
.input-group { text-align: left; margin-bottom: 1.5rem; }

.input-group label {
    display: block;
    margin-bottom: 0.5rem;
    color: var(--text-main);
    font-weight: 600;
    font-size: 0.85rem;
}

input {
    width: 100%;
    padding: 14px;
    border: 2px solid #f1f5f9;
    border-radius: 12px;
    background: #f8fafc;
    color: var(--dark-bg); /* Texto oscuro para contraste en fondo claro */
    font-size: 1rem;
    transition: all 0.2s ease;
}

input:focus {
    outline: none;
    border-color: var(--primary);
    background: #ffffff;
    box-shadow: 0 0 0 4px rgba(99, 102, 241, 0.1);
}

/* Botón con degradado suave */
.btn-login {
    width: 100%;
    padding: 14px;
    background: var(--dark-bg);
    color: white;
    border: none;
    border-radius: 12px;
    font-size: 1rem;
    font-weight: 700;
    cursor: pointer;
    transition: all 0.3s ease;
    margin-top: 1rem;
}

.btn-login:hover:not(:disabled) {
    background: var(--primary);
    transform: translateY(-2px);
    box-shadow: 0 10px 15px -3px rgba(99, 102, 241, 0.3);
}

.btn-login:disabled { opacity: 0.7; cursor: not-allowed; }

/* Mensaje de error más elegante */
.error-message {
    margin-top: 1.5rem;
    padding: 12px;
    background: #fef2f2;
    color: var(--error);
    border-radius: 10px;
    font-size: 0.85rem;
    font-weight: 600;
    border: 1px solid #fee2e2;
}

/* Animación de entrada para el error */
.slide-up-enter-active { transition: all 0.3s ease-out; }
.slide-up-enter-from { opacity: 0; transform: translateY(10px); }
</style>