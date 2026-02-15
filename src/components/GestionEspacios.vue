<script setup>
import { ref, onMounted } from 'vue'

const espacios = ref([])

// Estructura exacta basada en tus imágenes JSON
const nuevoEspacio = ref({
    id: '',
    nombre: '',
    ubicacion_planta: '',
    capacidad_max: 0,
    equipamiento: '',
    estado_operativo: 'true' // Almacenado como string según tu captura
})

const cargarEspacios = async () => {
    try {
        const respuesta = await fetch('http://100.52.46.68:3000/espacios')
        espacios.value = await respuesta.json()
    } catch (error) {
        console.error("Error al cargar espacios:", error);
    }
}

const guardarEspacio = async () => {
    // Cálculo manual del ID autoincremental para evitar el error 'null value in column id'
    const maxId = espacios.value.length > 0
        ? Math.max(...espacios.value.map(e => Number(e.id)))
        : 0;
    nuevoEspacio.value.id = (maxId + 1).toString();

    try {
        const respuesta = await fetch('http://100.52.46.68:3000/espacios', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(nuevoEspacio.value)
        });

        if (respuesta.ok) {
            alert(`Espacio "${nuevoEspacio.value.nombre}" registrado con ID: ${nuevoEspacio.value.id}`);
            // Resetear formulario
            nuevoEspacio.value = {
                id: '', nombre: '', ubicacion_planta: '',
                capacidad_max: 0, equipamiento: '', estado_operativo: 'true'
            };
            await cargarEspacios();
        } else {
            const errorData = await respuesta.json();
            alert("Error del servidor: " + (errorData.error || "No se pudo insertar"));
        }
    } catch (e) {
        alert("Error de red al conectar con la API");
    }
}

const borrarEspacio = async (id) => {
    if (confirm("¿Estás seguro de eliminar este espacio?")) {
        await fetch(`http://100.52.46.68:3000/espacios/${id}`, { method: 'DELETE' });
        cargarEspacios();
    }
}

onMounted(cargarEspacios);
</script>

<template>
    <div class="gestion-container">
        <h3>Gestión de Espacios (H6)</h3>

        <form @submit.prevent="guardarEspacio" class="form-grid">
            <input v-model="nuevoEspacio.nombre" placeholder="Nombre (ej. Aula 103)" required>
            <input v-model="nuevoEspacio.ubicacion_planta" placeholder="Planta (ej. Primera planta)" required>

            <div class="input-labeled">
                <label>Capacidad Máx.</label>
                <input v-model="nuevoEspacio.capacidad_max" type="number" required>
            </div>

            <select v-model="nuevoEspacio.estado_operativo" required>
                <option value="true">Operativo</option>
                <option value="false">No Operativo</option>
            </select>

            <input v-model="nuevoEspacio.equipamiento" placeholder="Equipamiento (ej. Pizarra, Proyector)"
                class="full-width">

            <button type="submit" class="btn-success">Registrar Espacio</button>
        </form>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Planta</th>
                        <th>Capacidad</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="es in espacios" :key="es.id">
                        <td>{{ es.id }}</td>
                        <td>{{ es.nombre }}</td>
                        <td>{{ es.ubicacion_planta }}</td>
                        <td>{{ es.capacidad_max }} pax</td>
                        <td>
                            <span :class="es.estado_operativo === 'true' ? 'status-ok' : 'status-err'">
                                {{ es.estado_operativo === 'true' ? 'Disponible' : 'Fuera de Servicio' }}
                            </span>
                        </td>
                        <td>
                            <button @click="borrarEspacio(es.id)" class="btn-danger">Borrar</button>
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
}

.full-width {
    grid-column: span 2;
}

.input-labeled {
    display: flex;
    flex-direction: column;
    text-align: left;
}

.input-labeled label {
    font-size: 0.7rem;
    color: #7f8c8d;
    margin-bottom: 2px;
    padding-left: 5px;
}

select,
input {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    color: #fff;
    /* Texto blanco solicitado */
    background: #2c3e50;
}

option {
    color: #fff;
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

.status-ok {
    color: #27ae60;
    font-weight: bold;
}

.status-err {
    color: #e74c3c;
    font-weight: bold;
}

.btn-danger {
    background: #e74c3c;
    color: white;
    border: none;
    padding: 6px 10px;
    border-radius: 4px;
    cursor: pointer;
}
</style>