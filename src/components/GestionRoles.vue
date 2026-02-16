<script setup>
import { ref, onMounted } from 'vue'

const roles = ref([])
const nuevoRol = ref({
    id: '',
    nombre: '',
    nivel_privilegio: 1,
    descripcion: ''
})
const rolEditando = ref(null)

const cargarRoles = async () => {
    try {
        const respuesta = await fetch('http://100.52.46.68:3000/roles')
        const data = await respuesta.json()
        // Ordenamos por ID para mantener la estabilidad visual
        roles.value = data.sort((a, b) => Number(a.id) - Number(b.id))
    } catch (error) {
        console.error("Error cargando roles:", error)
    }
}

const guardarRol = async () => {
    const maxId = roles.value.length > 0
        ? Math.max(...roles.value.map(r => Number(r.id)))
        : 0;
    nuevoRol.value.id = (maxId + 1).toString();

    const respuesta = await fetch('http://100.52.46.68:3000/roles', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(nuevoRol.value)
    });

    if (respuesta.ok) {
        nuevoRol.value = { id: '', nombre: '', nivel_privilegio: 1, descripcion: '' };
        await cargarRoles();
    }
}

const actualizarRol = async () => {
    const id = rolEditando.value.id;
    await fetch(`http://100.52.46.68:3000/roles/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(rolEditando.value)
    });
    rolEditando.value = null;
    await cargarRoles();
}

const borrarRol = async (id) => {
    if (confirm("¿Eliminar este rol? Asegúrate de que no haya usuarios asignados.")) {
        await fetch(`http://100.52.46.68:3000/roles/${id}`, { method: 'DELETE' });
        await cargarRoles();
    }
}

const iniciarEdicion = (rol) => { rolEditando.value = { ...rol }; }
const cancelarEdicion = () => { rolEditando.value = null; }

onMounted(cargarRoles);
</script>

<template>
    <div class="gestion-container">
        <h3>Gestión de Roles (H11)</h3>

        <form @submit.prevent="rolEditando ? actualizarRol() : guardarRol()" class="form-grid">
            <input v-if="!rolEditando" v-model="nuevoRol.nombre" placeholder="Nombre del Rol (ej. Editor)" required>
            <input v-else v-model="rolEditando.nombre" placeholder="Nombre" required>

            <div class="input-group">
                <label>Nivel Privilegio (1-3)</label>
                <input v-if="!rolEditando" v-model="nuevoRol.nivel_privilegio" type="number" min="1" max="3" required>
                <input v-else v-model="rolEditando.nivel_privilegio" type="number" min="1" max="3" required>
            </div>

            <input v-if="!rolEditando" v-model="nuevoRol.descripcion" placeholder="Descripción del nivel de acceso"
                class="full-width" required>
            <input v-else v-model="rolEditando.descripcion" placeholder="Descripción" class="full-width" required>

            <div class="form-actions">
                <button type="submit" class="btn-success">
                    {{ rolEditando ? 'Guardar Cambios' : 'Crear Rol' }}
                </button>
                <button v-if="rolEditando" type="button" @click="cancelarEdicion" class="btn-sec">Cancelar</button>
            </div>
        </form>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Rol</th>
                        <th>Nivel</th>
                        <th>Descripción</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="rol in roles" :key="rol.id">
                        <td>{{ rol.id }}</td>
                        <td><strong>{{ rol.nombre }}</strong></td>
                        <td>Nivel {{ rol.nivel_privilegio }}</td>
                        <td>{{ rol.descripcion }}</td>
                        <td class="td-actions">
                            <button @click="iniciarEdicion(rol)" class="btn-accent">Editar</button>
                            <button @click="borrarRol(rol.id)" class="btn-danger">Borrar</button>
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

.input-group {
    display: flex;
    flex-direction: column;
    text-align: left;
}

.input-group label {
    font-size: 0.7rem;
    color: #7f8c8d;
    margin-bottom: 2px;
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