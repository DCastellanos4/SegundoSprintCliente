<script setup>
import { ref, onMounted } from 'vue'

const cursos = ref([])
const profesores = ref([])
const etapas = ref([])
const turnos = ref([])
const alumnos = ref([])
const espacios = ref([])

const cursoVerAlumnos = ref(null)
const mostrarModalAlumnos = ref(false)

const nuevoCurso = ref({
    id: '', // Lo calcularemos antes de enviar
    nombre_curso: '',
    etapa_id: '',
    grupo: '',
    turno_id: '',
    anio_academico: '',
    tutor_id: '',
    aula_id: ''
})

const cargarDatos = async () => {
    try {
        const [resCur, resProf, resEt, resTur, resAlu, resEsp] = await Promise.all([
            fetch('http://100.52.46.68:3000/cursos'),
            fetch('http://100.52.46.68:3000/profesores'),
            fetch('http://100.52.46.68:3000/etapas'),
            fetch('http://100.52.46.68:3000/turnos'),
            fetch('http://100.52.46.68:3000/alumnos'),
            fetch('http://100.52.46.68:3000/espacios')
        ]);

        cursos.value = await resCur.json();
        profesores.value = await resProf.json();
        etapas.value = await resEt.json();
        turnos.value = await resTur.json();
        alumnos.value = await resAlu.json();
        espacios.value = await resEsp.json();
    } catch (error) {
        console.error("Error cargando datos:", error);
    }
}

const guardarCurso = async () => {
    // 1. Cálculo manual del ID (Solución al error de la imagen)
    // Buscamos el ID más alto y le sumamos 1. Si no hay cursos, empezamos en 1.
    const maxId = cursos.value.length > 0
        ? Math.max(...cursos.value.map(c => Number(c.id)))
        : 0;
    nuevoCurso.value.id = (maxId + 1).toString();

    // 2. Validación de Tutoría Única
    const tutorOcupado = cursos.value.find(c =>
        Number(c.tutor_id) === Number(nuevoCurso.value.tutor_id) &&
        c.anio_academico === nuevoCurso.value.anio_academico
    );

    if (tutorOcupado) {
        alert(`Advertencia: Este profesor ya es tutor en ${nuevoCurso.value.anio_academico}.`);
        return;
    }

    try {
        const respuesta = await fetch('http://100.52.46.68:3000/cursos', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(nuevoCurso.value)
        });

        if (respuesta.ok) {
            alert(`Curso creado exitosamente con ID: ${nuevoCurso.value.id}`);
            nuevoCurso.value = {
                id: '', nombre_curso: '', etapa_id: '', grupo: '',
                turno_id: '', anio_academico: '', tutor_id: '', aula_id: ''
            };
            await cargarDatos();
        } else {
            const errorMsg = await respuesta.text();
            alert("Error al insertar: " + errorMsg);
        }
    } catch (e) {
        alert("Error de red al conectar con la API");
    }
}

const verAlumnos = (curso) => {
    cursoVerAlumnos.value = curso;
    mostrarModalAlumnos.value = true;
};

const intentarBorrarCurso = async (id) => {
    const tieneAlumnos = alumnos.value.some(a => Number(a.curso_id) === Number(id));
    if (tieneAlumnos) {
        alert("No se puede borrar: El curso tiene alumnos vinculados.");
        return;
    }
    if (confirm("¿Borrar curso?")) {
        await fetch(`http://100.52.46.68:3000/cursos/${id}`, { method: 'DELETE' });
        cargarDatos();
    }
}

onMounted(cargarDatos);
</script>

<template>
    <div class="gestion-container">
        <h3>Gestión de Cursos (H5)</h3>

        <form @submit.prevent="guardarCurso" class="form-grid">
            <input v-model="nuevoCurso.nombre_curso" placeholder="Nombre (ej. 2º DAW)" required>
            <input v-model="nuevoCurso.grupo" placeholder="Grupo (ej. A)" required>
            <input v-model="nuevoCurso.anio_academico" placeholder="Año (ej. 2025-2026)" required>

            <select v-model="nuevoCurso.etapa_id" required>
                <option value="" disabled>Etapa</option>
                <option v-for="e in etapas" :key="e.id" :value="e.id">{{ e.nombre }}</option>
            </select>

            <select v-model="nuevoCurso.turno_id" required>
                <option value="" disabled>Turno</option>
                <option v-for="t in turnos" :key="t.id" :value="t.id">{{ t.nombre }}</option>
            </select>

            <select v-model="nuevoCurso.aula_id" required>
                <option value="" disabled>Espacio Asignado</option>
                <option v-for="es in espacios" :key="es.id" :value="es.id">{{ es.nombre }}</option>
            </select>

            <select v-model="nuevoCurso.tutor_id" required>
                <option value="" disabled>Tutor</option>
                <option v-for="p in profesores" :key="p.id" :value="p.id">{{ p.nombre }} {{ p.apellidos }}</option>
            </select>

            <button type="submit" class="btn-success">Registrar Curso</button>
        </form>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Curso/Grupo</th>
                        <th>Año</th>
                        <th>Tutor</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="c in cursos" :key="c.id">
                        <td>{{ c.id }}</td>
                        <td>{{ c.nombre_curso }} - {{ c.grupo }}</td>
                        <td>{{ c.anio_academico }}</td>
                        <td>{{profesores.find(p => p.id == c.nombre)?.nombre || 'S/T'}}</td>
                        <td>
                            <button @click="verAlumnos(c)" class="btn-accent">Alumnos</button>
                            <button @click="intentarBorrarCurso(c.id)" class="btn-danger">Borrar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <div v-if="mostrarModalAlumnos" class="modal-overlay">
            <div class="modal-box">
                <h4>Alumnos en {{ cursoVerAlumnos?.nombre_curso }}</h4>
                <div class="lista-scroll">
                    <ul v-if="alumnos.filter(a => Number(a.curso_id) === Number(cursoVerAlumnos?.id)).length > 0">
                        <li v-for="a in alumnos.filter(a => Number(a.curso_id) === Number(cursoVerAlumnos?.id))"
                            :key="a.id">
                            {{ a.nombre }} {{ a.apellidos }} ({{ a.nia }})
                        </li>
                    </ul>
                    <p v-else>No hay alumnos en este curso.</p>
                </div>
                <button @click="mostrarModalAlumnos = false" class="btn-sec">Cerrar</button>
            </div>
        </div>
    </div>
</template>

<style scoped>
.form-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 15px;
    background: #fff;
    padding: 20px;
    border-radius: 8px;
}

select,
input {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    color: #fff;
    /* Texto blanco solicitado */
}

option {
    color: #fff;
    /* Opciones blancas solicitadas */
}

.btn-success {
    grid-column: span 2;
    background: #27ae60;
    color: white;
    border: none;
    padding: 12px;
    cursor: pointer;
    border-radius: 4px;
    font-weight: bold;
}

.table-container {
    margin-top: 25px;
    background: white;
    padding: 15px;
    border-radius: 8px;
    color: #333;
}

table {
    width: 100%;
    border-collapse: collapse;
}

th,
td {
    padding: 12px;
    text-align: left;
    border-bottom: 1px solid #eee;
}

.btn-accent {
    background: #f39c12;
    color: white;
    margin-right: 8px;
    border: none;
    padding: 6px 10px;
    border-radius: 4px;
    cursor: pointer;
}

.btn-danger {
    background: #e74c3c;
    color: white;
    border: none;
    padding: 6px 10px;
    border-radius: 4px;
    cursor: pointer;
}

.btn-sec {
    background: #95a5a6;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
    margin-top: 15px;
}

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
    padding: 30px;
    border-radius: 12px;
    max-width: 450px;
    width: 90%;
    color: #333;
    text-align: center;
}
</style>