<script setup>
import { ref } from 'vue'

const emit = defineEmits(['login-exitoso'])

const nombreUsuario = ref('')
const passwordUsuario = ref('')
const errorLogin = ref(false)

const realizarLogin = async () => {
    try {
        const respuesta = await fetch('http://100.52.46.68:3000/usuarios');
        const usuarios = await respuesta.json();

        const encontrado = usuarios.find(u =>
            u.login === nombreUsuario.value && u.password_hash === passwordUsuario.value
        );

        if (encontrado) {
            emit('login-exitoso', encontrado);
        } else {
            errorLogin.value = true;
        }
    } catch (e) { console.error(e); }
};
</script>

<template>
    <div class="login-container">
        <div class="login-box">
            <div class="login-header">
                <h2>Escarlatti-Gest</h2>
                <p>Gestión de Incidencias y Aulas</p>
            </div>

            <form @submit.prevent="realizarLogin">
                <div class="input-group">
                    <label>Usuario</label>
                    <input v-model="nombreUsuario" placeholder="Introduce tu usuario" required>
                </div>

                <div class="input-group">
                    <label>Contraseña</label>
                    <input v-model="passwordUsuario" type="password" placeholder="••••••••" required>
                </div>

                <button type="submit" class="btn-login">Entrar al Sistema</button>
            </form>

            <div v-if="errorLogin" class="error-message">
                ⚠️ Credenciales incorrectas. Inténtalo de nuevo.
            </div>
        </div>
    </div>
</template>

<style scoped>
/* Contenedor para centrar el login en toda la pantalla */
.login-container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 80vh;
    /* Ajustado para que no choque con el margen de App.vue */
}

.login-box {
    background: white;
    padding: 40px;
    border-radius: 12px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 400px;
    text-align: center;
}

.login-header {
    margin-bottom: 30px;
}

.logo-placeholder {
    font-size: 3rem;
    margin-bottom: 10px;
}

h2 {
    color: #2c3e50;
    margin: 0;
    font-size: 1.8rem;
}

.login-header p {
    color: #7f8c8d;
    font-size: 0.9rem;
    margin-top: 5px;
}

/* Grupos de Input */
.input-group {
    text-align: left;
    margin-bottom: 20px;
}

.input-group label {
    display: block;
    font-weight: 600;
    color: #34495e;
    margin-bottom: 8px;
    font-size: 0.85rem;
}

input {
    width: 100%;
    padding: 12px;
    border: 1px solid #ddd;
    border-radius: 6px;
    box-sizing: border-box;
    font-size: 1rem;
    transition: border-color 0.3s;
    color: #fff;
}

input:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 5px rgba(52, 152, 219, 0.2);
}

/* Botón Principal */
.btn-login {
    width: 100%;
    padding: 14px;
    background-color: #2c3e50;
    /* Color corporativo oscuro */
    color: white;
    border: none;
    border-radius: 6px;
    font-size: 1rem;
    font-weight: bold;
    cursor: pointer;
    transition: background 0.3s;
}

.btn-login:hover {
    background-color: #34495e;
}

/* Mensaje de Error */
.error-message {
    margin-top: 20px;
    padding: 10px;
    background-color: #fdeaea;
    color: #e74c3c;
    border-radius: 4px;
    font-size: 0.85rem;
    font-weight: 600;
}
</style>