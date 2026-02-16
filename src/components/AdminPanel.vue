<script setup>
/**
 * COMPONENTE: AdminPanel.vue
 * gestiona la navegacion interna de cada usuario
 */
import { ref } from 'vue'
import Profesores from './GestionProfesores.vue'
import Alumnos from './GestionAlumnos.vue'
import Cursos from './GestionCursos.vue'
import Espacios from './GestionEspacios.vue' // H6: Lugares para reservas e incidencias
import Etapas from './GestionEtapas.vue'
import Turnos from './GestionTurnos.vue'
import Departamento from './GestionDepartamentos.vue'
import Roles from './GestionRoles.vue'

// recibimos el usuario para saber que rol tiene y mostrarle sus opciones
const props = defineProps(['usuario']);
// evento para poder hacer el logout
const emit = defineEmits(['logout']);

// Estado reactivo para saber qué componente hijo renderizar
const seccionActual = ref('menu');

// cambiamos el valor de la ref para que el v-else-if haga el cambio del render
const irAMenu = () => seccionActual.value = 'menu';
const irAProfesores = () => seccionActual.value = 'profesores';
const irAAlumnos = () => seccionActual.value = 'alumnos';
const irACursos = () => seccionActual.value = 'cursos';
const irAEspacios = () => seccionActual.value = 'espacios';
const irAEtapas = () => seccionActual.value = 'etapas';
const irAturnos = () => seccionActual.value = 'turnos';
const irAdepartamentos = () => seccionActual.value = 'departamentos';
const irAroles = () => seccionActual.value = 'roles';

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
                    <button @click="irAEspacios">Espacios (H6)</button>
                    <button @click="irAEtapas">Etapas (H8)</button>
                    <button @click="irAturnos">Turnos (H9)</button>
                    <button @click="irAdepartamentos">Departamentos (H10)</button>
                    <button @click="irAroles">Roles (H11)</button>
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
        <div v-else-if="seccionActual === 'etapas'">
            <Etapas />
        </div>
        <div v-else-if="seccionActual === 'turnos'">
            <Turnos />
        </div>
        <div v-else-if="seccionActual === 'departamentos'">
            <Departamento />
        </div>
        <div v-else-if="seccionActual === 'roles'">
            <Roles />
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

button:hover {
    opacity: 0.9;
}

.btn-primary {
    background-color: #3498db;
    color: white;
}

.btn-accent {
    background-color: #f39c12;
    color: white;
}

.btn-success {
    background-color: #27ae60;
    color: white;
}

.btn-danger {
    background-color: #e74c3c;
    color: white;
}

.logout-btn {
    background-color: #666;
    color: white;
}

.admin-buttons-container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    margin-top: 10px;
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