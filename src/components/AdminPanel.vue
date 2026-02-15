<script setup>
import { ref } from 'vue'
import Profesores from './GestionProfesores.vue'
import Alumnos from './GestionAlumnos.vue'
import Cursos from './GestionCursos.vue'
import Espacios from './GestionEspacios.vue' // 1. Importar el nuevo componente H6

const props = defineProps(['usuario']);
const emit = defineEmits(['logout']);

// Variable para controlar qué sección vemos
const seccionActual = ref('menu');

// 2. Funciones de navegación actualizadas
const irAMenu = () => seccionActual.value = 'menu';
const irAProfesores = () => seccionActual.value = 'profesores';
const irAAlumnos = () => seccionActual.value = 'alumnos';
const irACursos = () => seccionActual.value = 'cursos';
const irAEspacios = () => seccionActual.value = 'espacios'; // Nueva función H6

// Funciones para las acciones de servicios
const crearIncidencia = () => console.log("H2: Abriendo formulario...");
const reservarAula = () => console.log("H2: Abriendo reservas...");
const resolverIncidencia = () => console.log("H2: Listado TIC...");
</script>

<template>
    <div class="panel-container">
        <header>
            <h3>Bienvenid@, {{ usuario.login }}</h3>
            <div class="header-buttons">
                <button v-if="seccionActual !== 'menu'" @click="irAMenu" class="btn-primary">Volver al Panel</button>
                <button @click="$emit('logout')" class="logout-btn">Cerrar Sesión</button>
            </div>
        </header>

        <div v-if="seccionActual === 'menu'" class="acciones-grid">
            <section v-if="usuario">
                <h4>Servicios Generales</h4>
                <button @click="crearIncidencia()" class="btn-primary">Crear Incidencia</button>
            </section>

            <section v-if="[1, 2, 4].includes(Number(usuario.rol_id))">
                <h4>Gestión de Espacios</h4>
                <button @click="reservarAula" class="btn-accent">Reservar Aula</button>
            </section>

            <section v-if="[1, 4].includes(Number(usuario.rol_id))">
                <h4>Mantenimiento TIC</h4>
                <button @click="resolverIncidencia" class="btn-success">Resolver Incidencias</button>
            </section>

            <section v-if="Number(usuario.rol_id) === 1">
                <h4>Administración del Centro</h4>
                <div class="admin-buttons-container">
                    <button @click="irAProfesores">Profesores (H3)</button>
                    <button @click="irAAlumnos">Alumnos (H4)</button>
                    <button @click="irACursos">Cursos (H5)</button>
                    <button @click="irAEspacios">Gestionar Espacios (H6)</button>
                </div>
                <p class="admin-note">Tienes acceso total al sistema.</p>
            </section>
        </div>

        <div v-else-if="seccionActual === 'profesores'">
            <Profesores />
        </div>
        <div v-else-if="seccionActual === 'alumnos'">
            <Alumnos />
        </div>
        <div v-else-if="seccionActual === 'cursos'">
            <Cursos />
        </div>
        <div v-else-if="seccionActual === 'espacios'">
            <Espacios />
        </div>

    </div>
</template>

<style scoped>
.panel-container {
    background: #eee;
    padding: 20px;
    border-radius: 8px;
    width: 100%;
    max-width: 600px;
    color: #333;
}

header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    border-bottom: 2px solid #ddd;
    padding-bottom: 10px;
}

.header-buttons {
    display: flex;
    gap: 10px;
}

.acciones-grid {
    display: grid;
    gap: 20px;
}

section {
    border-top: 1px solid #ccc;
    padding-top: 15px;
}

h4 {
    font-size: 0.85rem;
    color: #555;
    margin-bottom: 12px;
    text-transform: uppercase;
    letter-spacing: 1px;
}

button {
    padding: 10px 16px;
    font-weight: bold;
    cursor: pointer;
    border-radius: 4px;
    border: none;
    transition: opacity 0.2s;
}

button:hover { opacity: 0.9; }

.btn-primary { background-color: #3498db; color: white; }
.btn-accent { background-color: #f39c12; color: white; }
.btn-success { background-color: #27ae60; color: white; }
.btn-danger { background-color: #e74c3c; color: white; }
.logout-btn { background-color: #666; color: white; }

/* Contenedor Grid para Admin */
.admin-buttons-container {
    display: grid;
    grid-template-columns: 1fr 1fr; /* Mantiene el diseño de 2 columnas */
    gap: 10px;
    margin-top: 10px;
}
.boton-admin{
    background-color: #333;
}

/* El botón de espacios (danger) se coloca al final */
.admin-buttons-container .btn-danger {
    grid-column: span 2;
    margin-top: 5px;
}

.admin-buttons-container button:not(.btn-danger) {
    background-color: #333;
    color: white;
}

.admin-note {
    font-size: 0.75rem;
    color: #7f8c8d;
    font-style: italic;
    margin-top: 10px;
}

section:last-of-type {
    margin-top: 10px;
    padding-top: 20px;
    border-top: 2px dashed #ccc;
}
</style>