<script setup>
import { ref, onMounted } from 'vue'

const alumnos = ref([])
const cursos = ref([])

const mostrarModalBorrado = ref(false)
const alumnoParaBorrar = ref(null)

// Ajustado exactamente a lo que tu API espera (según tus imágenes)
const nuevoAlumno = ref({
    nia: '',
    nombre: '',
    apellidos: '',
    curso_id: '',
    correo_electronico: '',
    tutor_legal_contacto: '',
    estado_id: 1
})

const cargarDatos = async () => {
    try {
        const [resAlu, resCur] = await Promise.all([
            fetch('http://100.52.46.68:3000/alumnos'),
            fetch('http://100.52.46.68:3000/cursos')
        ]);
        alumnos.value = await resAlu.json();
        cursos.value = await resCur.json();
    } catch (error) {
        console.error("Error al cargar datos:", error);
    }
}

const guardarAlumno = async () => {
    // Validación de duplicados por NIA o Correo
    const existe = alumnos.value.find(a =>
        a.nia === nuevoAlumno.value.nia ||
        a.correo_electronico === nuevoAlumno.value.correo_electronico
    );

    if (existe) {
        alert("Error: Ya existe un alumno con ese NIA o correo electrónico.");
        return;
    }

    try {
        const respuesta = await fetch('http://100.52.46.68:3000/alumnos', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(nuevoAlumno.value)
        });

        if (respuesta.ok) {
            // Resetear formulario
            nuevoAlumno.value = {
                nia: '', nombre: '', apellidos: '',
                curso_id: '', correo_electronico: '',
                tutor_legal_contacto: '', estado_id: 1
            };
            await cargarDatos();
            alert("Alumno insertado con éxito");
        } else {
            const errorText = await respuesta.text();
            alert("Error en el servidor: " + errorText);
        }
    } catch (error) {
        alert("Error de conexión al insertar");
    }
}

const ejecutarBorrado = async () => {
    try {
        await fetch(`http://100.52.46.68:3000/alumnos/${alumnoParaBorrar.value.nia}`, {
            method: 'DELETE'
        });
        mostrarModalBorrado.value = false;
        cargarDatos();
    } catch (error) {
        console.error("Error:", error);
    }
}

onMounted(cargarDatos);
</script>

<template>
    <div class="gestion-container">
        <h3>Gestión de Alumnos (H4)</h3>

        <form @submit.prevent="guardarAlumno" class="form-grid">
            <input v-model="nuevoAlumno.nia" placeholder="NIA (Ej: 2026002)" required>
            <input v-model="nuevoAlumno.nombre" placeholder="Nombre" required>
            <input v-model="nuevoAlumno.apellidos" placeholder="Apellidos" required>
            <input v-model="nuevoAlumno.correo_electronico" type="text" placeholder="Correo Electrónico" required>
            <input v-model="nuevoAlumno.tutor_legal_contacto" placeholder="Email Tutor Legal" required>

            <select v-model="nuevoAlumno.curso_id" required>
                <option value="" disabled>Seleccione Curso</option>
                <option v-for="c in cursos" :key="c.id" :value="c.id">
                    {{ c.nombre_curso }}
                </option>
            </select>

            <div class="checkbox-container">
                <label>
                    <input type="checkbox" :checked="Number(nuevoAlumno.estado_id) === 1"
                        @change="nuevoAlumno.estado_id = $event.target.checked ? 1 : 2">
                    <span v-if="Number(nuevoAlumno.estado_id) === 1">Alumno Activo</span>
                    <span v-else>Alumno Inactivo</span>
                </label>
            </div>

            <button type="submit" class="btn-success">Registrar Alumno</button>
        </form>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>NIA</th>
                        <th>Alumno</th>
                        <th>Curso</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="a in alumnos" :key="a.id">
                        <td>{{ a.nia }}</td>
                        <td>{{ a.nombre }} {{ a.apellidos }}</td>
                        <td>{{cursos.find(c => c.id == a.curso_id)?.nombre_curso || 'N/A'}}</td>
                        <td>
                            <span v-if="Number(a.estado_id) === 1" class="status-active">Activo</span>
                            <span v-else class="status-inactive">Inactivo</span>
                        </td>
                        <td>
                            <button @click="alumnoParaBorrar = a; mostrarModalBorrado = true"
                                class="btn-danger">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <div v-if="mostrarModalBorrado" class="modal-overlay">
            <div class="modal-box">
                <h4>¿Eliminar Alumno?</h4>
                <p>¿Estás seguro de eliminar a <strong>{{ alumnoParaBorrar.nombre }}</strong>?</p>
                <div class="modal-actions">
                    <button @click="ejecutarBorrado" class="btn-danger">Confirmar</button>
                    <button @click="mostrarModalBorrado = false" class="btn-sec">Cancelar</button>
                </div>
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
    /* Cambiado a oscuro para que sea legible */
}

.checkbox-container {
    display: flex;
    align-items: center;
    grid-column: span 1;
    padding-left: 10px;
    color: #333;
}

.btn-success {
    grid-column: span 2;
    background: #27ae60;
    color: white;
    border: none;
    padding: 12px;
    cursor: pointer;
    border-radius: 4px;
}

.status-active {
    color: #27ae60;
    font-weight: bold;
}

.status-inactive {
    color: #e74c3c;
    font-weight: bold;
}

.table-container {
    margin-top: 30px;
    background: white;
    padding: 10px;
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
    text-align: center;
    color: #333;
}

.modal-actions {
    display: flex;
    gap: 10px;
    justify-content: center;
    margin-top: 20px;
}

.btn-danger {
    background: #e74c3c;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 4px;
    cursor: pointer;
}

.btn-sec {
    background: #95a5a6;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 4px;
    cursor: pointer;
}
</style>