<script setup>
import { ref, onMounted } from 'vue'

const departamentos = ref([])
const profesores = ref([])

// Campos ajustados a tu captura: id, nombre, ubicacion, correo_contacto
const nuevoDep = ref({
    id: '',
    nombre: '',
    ubicacion: '',
    correo_contacto: ''
})
const depEditando = ref(null)

const cargarDatos = async () => {
    try {
        const [resDep, resProf] = await Promise.all([
            fetch('http://100.52.46.68:3000/departamentos'),
            fetch('http://100.52.46.68:3000/profesores')
        ]);

        const dataDep = await resDep.json();
        // Ordenamos por ID para que la tabla sea estable
        departamentos.value = dataDep.sort((a, b) => Number(a.id) - Number(b.id));
        profesores.value = await resProf.json();
    } catch (error) {
        console.error("Error cargando departamentos:", error);
    }
}

const guardarDepartamento = async () => {
    const maxId = departamentos.value.length > 0
        ? Math.max(...departamentos.value.map(d => Number(d.id)))
        : 0;
    nuevoDep.value.id = (maxId + 1).toString();

    const respuesta = await fetch('http://100.52.46.68:3000/departamentos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(nuevoDep.value)
    });

    if (respuesta.ok) {
        nuevoDep.value = { id: '', nombre: '', ubicacion: '', correo_contacto: '' };
        await cargarDatos();
    }
}

const actualizarDepartamento = async () => {
    const id = depEditando.value.id;
    await fetch(`http://100.52.46.68:3000/departamentos/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(depEditando.value)
    });
    depEditando.value = null;
    await cargarDatos();
}

const borrarDepartamento = async (id) => {
    const tieneProfesores = profesores.value.some(p => Number(p.departamento_id) === Number(id));

    if (tieneProfesores) {
        alert("Acción bloqueada: Hay profesores asignados a este departamento.");
        return;
    }

    if (confirm("¿Eliminar este departamento?")) {
        await fetch(`http://100.52.46.68:3000/departamentos/${id}`, { method: 'DELETE' });
        await cargarDatos();
    }
}

const iniciarEdicion = (dep) => { depEditando.value = { ...dep }; }
const cancelarEdicion = () => { depEditando.value = null; }

onMounted(cargarDatos);
</script>

<template>
    <div class="gestion-container">
        <h3>Gestión de Departamentos (H10)</h3>

        <form @submit.prevent="depEditando ? actualizarDepartamento() : guardarDepartamento()" class="form-grid">
            <input v-if="!depEditando" v-model="nuevoDep.nombre" placeholder="Nombre (ej. Informática)" required>
            <input v-else v-model="depEditando.nombre" placeholder="Nombre" required>

            <input v-if="!depEditando" v-model="nuevoDep.ubicacion" placeholder="Ubicación (ej. Edificio A, planta 2)"
                required>
            <input v-else v-model="depEditando.ubicacion" placeholder="Ubicación" required>

            <input v-if="!depEditando" v-model="nuevoDep.correo_contacto" placeholder="Correo (ej. info@escarlatti.es)"
                class="full-width" required>
            <input v-else v-model="depEditando.correo_contacto" placeholder="Correo" class="full-width" required>

            <div class="form-actions">
                <button type="submit" class="btn-success">
                    {{ depEditando ? 'Actualizar' : 'Registrar Departamento' }}
                </button>
                <button v-if="depEditando" type="button" @click="cancelarEdicion" class="btn-sec">Cancelar</button>
            </div>
        </form>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Ubicación</th>
                        <th>Contacto</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="d in departamentos" :key="d.id">
                        <td>{{ d.id }}</td>
                        <td>{{ d.nombre }}</td>
                        <td>{{ d.ubicacion }}</td>
                        <td>{{ d.correo_contacto }}</td>
                        <td class="td-actions">
                            <button @click="iniciarEdicion(d)" class="btn-accent">Editar</button>
                            <button @click="borrarDepartamento(d.id)" class="btn-danger">Borrar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
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
    margin-bottom: 20px;
}

.full-width {
    grid-column: span 2;
}

.form-actions {
    grid-column: span 2;
    display: flex;
    gap: 10px;
}

input {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    color: #fff;
    background: #2c3e50;
}

.btn-success {
    background: #27ae60;
    color: white;
    border: none;
    padding: 12px;
    border-radius: 4px;
    font-weight: bold;
    flex: 2;
    cursor: pointer;
}

.btn-sec {
    background: #95a5a6;
    color: white;
    border: none;
    padding: 12px;
    border-radius: 4px;
    flex: 1;
    cursor: pointer;
}

.btn-accent {
    background: #f39c12;
    color: white;
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

.td-actions {
    display: flex;
    gap: 8px;
}

.table-container {
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
</style>