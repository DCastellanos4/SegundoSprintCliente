<script setup>
/**
 * COMPONENTE: GestionAlumnos.vue
 * Descripción: Módulo para el CRUD de alumnos. 
 * Incluye gestión de estados (Activo/Inactivo) y validación de duplicados.
 */
import { ref, onMounted } from 'vue'

const listaAlumnos = ref([])
const listaCursos = ref([])
const cargando = ref(false)

// Control de UI para el borrado 
const mostrarModalBorrado = ref(false)
const alumnoSeleccionado = ref(null)

// Modelo para el formulario
const modeloAlumno = ref({
    nia: '',
    nombre: '',
    apellidos: '',
    curso_id: '',
    correo_electronico: '',
    tutor_legal_contacto: '',
    estado_id: 1
})

/**
 * Función para traer datos de la API.
 * Usamos Promise.all para cargar alumnos y cursos en paralelo y ganar velocidad.
 */
const cargarInformacion = async () => {
    cargando.value = true
    try {
        const [respuestaAlumnos, respuestaCursos] = await Promise.all([
            fetch('http://100.52.46.68:3000/alumnos'),
            fetch('http://100.52.46.68:3000/cursos')
        ]);

        const datosBrutos = await respuestaAlumnos.json();

        /**
         * ordenacion por NIA de alumnos
         */
        listaAlumnos.value = datosBrutos.sort((a, b) => {
            return a.nia.localeCompare(b.nia, undefined, { numeric: true });
        });

        listaCursos.value = await respuestaCursos.json();
    } catch (error) {
        console.error("Fallo al sincronizar con la API:", error);
    } finally {
        cargando.value = false
    }
}

/**
 * Alternar entre activo e inactivo
 */
const conmutarEstado = async (alumno) => {
    // Si es 1 pasa a 2, si es cualquier otra cosa (inactivo) vuelve a 1
    let nuevoEstado;

    if (Number(alumno.estado_id) === 1) {
        nuevoEstado = 2;
    } else {
        nuevoEstado = 1;
    }
    // Ahora ya podemos usar 'nuevoEstado' para actualizar el objeto
    const datosActualizados = { ...alumno, estado_id: nuevoEstado };


    try {
        const res = await fetch(`http://100.52.46.68:3000/alumnos/${alumno.nia}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(datosActualizados)
        });

        if (res.ok) await cargarInformacion();
        else alert("La API rechazó el cambio de estado.");

    } catch (error) {
        console.error("Error en la conexión PUT:", error);
    }
}

/**
 * registrar un alumno que no tenga mismo NIA y email
 */
const registrarAlumno = async () => {
    // validacion del lado del cliente para ahorrar peticiones a la API
    const duplicado = listaAlumnos.value.find(a =>
        a.nia === modeloAlumno.value.nia ||
        a.correo_electronico === modeloAlumno.value.correo_electronico
    );

    if (duplicado) {
        alert("¡Ojo! Ese NIA o correo ya están registrados en el sistema.");
        return;
    }

    try {
        const res = await fetch('http://100.52.46.68:3000/alumnos', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(modeloAlumno.value)
        });

        if (res.ok) {
            // Limpiamos el formulario reseteando el objeto
            modeloAlumno.value = {
                nia: '', nombre: '', apellidos: '',
                curso_id: '', correo_electronico: '',
                tutor_legal_contacto: '', estado_id: 1
            };
            await cargarInformacion();
            alert("Alumno dado de alta correctamente.");
        }
    } catch (error) {
        alert("No se pudo conectar con el servidor de base de datos.");
    }
}

/**
 * metodo de borrado para alumnos
 */
const borrarDefinitivamente = async () => {
    try {
        const res = await fetch(`http://100.52.46.68:3000/alumnos/${alumnoSeleccionado.value.nia}`, {
            method: 'DELETE'
        });

        if (res.ok) {
            mostrarModalBorrado.value = false;
            await cargarInformacion();
        }
    } catch (error) {
        console.error("Error al eliminar registro:", error);
    }
}

// cuando se carga, enseñamos la informacion de los alumnos
onMounted(cargarInformacion);
</script>

<template>
    <div class="gestion-container">
        <header class="seccion-header">
            <h3>Gestión de Alumnado (H4)</h3>
        </header>

        <form @submit.prevent="registrarAlumno" class="form-grid">
            <input v-model="modeloAlumno.nia" placeholder="NIA (Ej: 2026002)" required>
            <input v-model="modeloAlumno.nombre" placeholder="Nombre" required>
            <input v-model="modeloAlumno.apellidos" placeholder="Apellidos" required>
            <input v-model="modeloAlumno.correo_electronico" type="email" placeholder="Email institucional" required>
            <input v-model="modeloAlumno.tutor_legal_contacto" placeholder="Email Tutor/a Legal" required>

            <select v-model="modeloAlumno.curso_id" required>
                <option value="" disabled>Seleccionar Curso...</option>
                <option v-for="curso in listaCursos" :key="curso.id" :value="curso.id">
                    {{ curso.nombre_curso }}
                </option>
            </select>

            <button type="submit" class="btn-success">Añadir alumno</button>
        </form>

        <div class="table-container">
            <div v-if="cargando" class="loader">Sincronizando datos...</div>
            <table v-else>
                <thead>
                    <tr>
                        <th>NIA</th>
                        <th>Nombre Completo</th>
                        <th>Curso Asignado</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="alumno in listaAlumnos" :key="alumno.id">
                        <td><strong>{{ alumno.nia }}</strong></td>
                        <td>{{ alumno.nombre }} {{ alumno.apellidos }}</td>
                        <td>{{listaCursos.find(c => c.id == alumno.curso_id)?.nombre_curso || 'Sin curso'}}</td>
                        <td>
                            <span :class="Number(alumno.estado_id) === 1 ? 'status-active' : 'status-inactive'">
                                <!-- if's compactos -->
                                {{ Number(alumno.estado_id) === 1 ? 'ACTIVO' : 'INACTIVO' }}
                            </span>
                        </td>
                        <td class="td-actions">
                            <button @click="conmutarEstado(alumno)"
                                :class="Number(alumno.estado_id) === 1 ? 'btn-warn' : 'btn-ok'">
                                {{ Number(alumno.estado_id) === 1 ? 'Suspender' : 'Reactivar' }}
                            </button>

                            <button @click="alumnoSeleccionado = alumno; mostrarModalBorrado = true"
                                class="btn-danger">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <div v-if="mostrarModalBorrado" class="modal-overlay">
            <div class="modal-box">
                <h4>Atención: Acción Irreversible</h4>
                <p>¿Realmente deseas eliminar a <strong>{{ alumnoSeleccionado.nombre }}</strong> de la aplicación?</p>
                <div class="modal-actions">
                    <button @click="borrarDefinitivamente" class="btn-danger">Sí, eliminar</button>
                    <button @click="mostrarModalBorrado = false" class="btn-sec">Cancelar</button>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
/* Contenedores y Layout */
.seccion-header {
    display: flex;
    align-items: center;
    gap: 15px;
    margin-bottom: 20px;
}

.badge {
    background: #34495e;
    color: #fff;
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 0.7rem;
}

.form-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 15px;
    background: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

/* Inputs con estilo corporativo oscuro (Solicitado por el usuario) */
select,
input {
    padding: 12px;
    border: 1px solid #ddd;
    border-radius: 6px;
    color: #fff;
    background: #2c3e50;
    transition: border-color 0.3s;
}

input:focus {
    border-color: #3498db;
    outline: none;
}

.btn-success {
    grid-column: span 2;
    background: #27ae60;
    color: white;
    border: none;
    padding: 14px;
    cursor: pointer;
    border-radius: 6px;
    font-weight: bold;
}

/* Estados Visuales */
.status-active {
    color: #27ae60;
    font-weight: 800;
    font-size: 0.85rem;
}

.status-inactive {
    color: #e74c3c;
    font-weight: 800;
    font-size: 0.85rem;
}

.td-actions {
    display: flex;
    gap: 8px;
}

/* Botonería de la tabla */
.btn-warn {
    background: #f39c12;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.8rem;
}

.btn-ok {
    background: #27ae60;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.8rem;
}

.btn-danger {
    background: #e74c3c;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.8rem;
}

.btn-sec {
    background: #95a5a6;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 4px;
    cursor: pointer;
}

.table-container {
    margin-top: 30px;
    background: white;
    padding: 15px;
    border-radius: 8px;
    color: #333;
    overflow-x: auto;
}

table {
    width: 100%;
    border-collapse: collapse;
}

th,
td {
    padding: 14px;
    text-align: left;
    border-bottom: 1px solid #f0f0f0;
}

th {
    background: #f8f9fa;
    color: #7f8c8d;
    font-size: 0.8rem;
    text-transform: uppercase;
}

/* Capa de Modal */
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
}

.modal-box {
    background: white;
    padding: 40px;
    border-radius: 12px;
    text-align: center;
    color: #333;
    max-width: 400px;
}

.modal-actions {
    display: flex;
    gap: 15px;
    justify-content: center;
    margin-top: 25px;
}

.loader {
    padding: 40px;
    text-align: center;
    color: #95a5a6;
    font-style: italic;
}
</style>