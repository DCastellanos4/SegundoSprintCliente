<script setup>
/**
 * COMPONENTE: GestionEspacios.vue
 * Descripción: Mantenimiento de aulas y salas del centro 
 */
import { ref, onMounted } from 'vue'

const listaEspacios = ref([])

const modeloEspacio = ref({
    id: '',
    nombre: '',
    ubicacion_planta: '',
    capacidad_max: 0,
    equipamiento: '',
    estado_operativo: 'true' // Por defecto son disponibles
})

/**
 * Trae los datos de la API y los ordena para que la tabla sea estable.
 */
const cargarDatos = async () => {
    try {
        const respuesta = await fetch('http://100.52.46.68:3000/espacios')
        const datos = await respuesta.json()
        
        // ORDENAMIENTO: Ordenamos por ID para que no cambien de fila al editarlos
        listaEspacios.value = datos.sort((a, b) => Number(a.id) - Number(b.id))
    } catch (error) {
        console.error("Fallo al conectar con la API de espacios:", error);
    }
}

/**
 * Alternar Operatividad
 * Cambia entre 'true' (Disponible) y 'false' (Inacesible).
 */
const conmutarOperatividad = async (espacio) => {
    let nuevoEstado;

    if (espacio.estado_operativo === 'true') {
        nuevoEstado = 'false';
    } else {
        nuevoEstado = 'true';
    }

    const espacioActualizado = {
        ...espacio,
        estado_operativo: nuevoEstado
    };

    try {
        const respuesta = await fetch(`http://100.52.46.68:3000/espacios/${espacio.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(espacioActualizado)
        });

        if (respuesta.ok) {
            await cargarDatos(); // Refrescamos la lista para ver el cambio de color
        }
    } catch (error) {
        console.error("Error al actualizar estado del aula:", error);
    }
}

const guardarNuevoEspacio = async () => {
    //ERROR BBDD:
    // Cálculo manual del ID 
    const maxId = listaEspacios.value.length > 0
        ? Math.max(...listaEspacios.value.map(e => Number(e.id)))
        : 0;
    modeloEspacio.value.id = (maxId + 1).toString();

    try {
        const respuesta = await fetch('http://100.52.46.68:3000/espacios', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(modeloEspacio.value)
        });

        if (respuesta.ok) {
            alert(`Aula "${modeloEspacio.value.nombre}" registrada correctamente.`);
            // Reset del formulario
            modeloEspacio.value = {
                id: '', nombre: '', ubicacion_planta: '',
                capacidad_max: 0, equipamiento: '', estado_operativo: 'true'
            };
            await cargarDatos();
        }
    } catch (e) {
        alert("Error de red al registrar espacio.");
    }
}

const borrarRegistro = async (id) => {
    if (confirm("¿Seguro que quieres eliminar este espacio? Se borrarán sus datos asociados.")) {
        await fetch(`http://100.52.46.68:3000/espacios/${id}`, { method: 'DELETE' });
        await cargarDatos();
    }
}

onMounted(cargarDatos);
</script>

<template>
    <div class="gestion-container">
        <header class="seccion-header">
            <h3>Gestión de Espacios e Instalaciones (H6)</h3>
        </header>

        <form @submit.prevent="guardarNuevoEspacio" class="form-grid">
            <input v-model="modeloEspacio.nombre" placeholder="Nombre (ej. Aula 103)" required>
            <input v-model="modeloEspacio.ubicacion_planta" placeholder="Planta (ej. Segunda planta)" required>

            <div class="input-labeled">
                <label>Capacidad Máxima (Aforo)</label>
                <input v-model="modeloEspacio.capacidad_max" type="number" required>
            </div>

            <select v-model="modeloEspacio.estado_operativo" required>
                <option value="true">Operativo</option>
                <option value="false">No Operativo</option>
            </select>

            <input v-model="modeloEspacio.equipamiento" placeholder="Equipamiento (ej. Proyector, 20 PCs, AC)" class="full-width">

            <button type="submit" class="btn-success">Dar de Alta Espacio</button>
        </form>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Espacio</th>
                        <th>Planta</th>
                        <th>Aforo</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="es in listaEspacios" :key="es.id">
                        <td><strong>#{{ es.id }}</strong></td>
                        <td>{{ es.nombre }}</td>
                        <td>{{ es.ubicacion_planta }}</td>
                        <td>{{ es.capacidad_max }} pax</td>
                        <td>
                            <span :class="es.estado_operativo === 'true' ? 'status-ok' : 'status-err'">
                                {{ es.estado_operativo === 'true' ? 'Disponible' : 'Inaccesible ' }}
                            </span>
                        </td>
                        <td class="td-actions">
                            <button @click="conmutarOperatividad(es)" 
                                    :class="es.estado_operativo === 'true' ? 'btn-warn' : 'btn-ok'">
                                {{ es.estado_operativo === 'true' ? 'Desactivar' : 'Activar' }}
                            </button>
                            
                            <button @click="borrarRegistro(es.id)" class="btn-danger">Borrar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<style scoped>
.seccion-header { margin-bottom: 20px; border-left: 5px solid #3498db; padding-left: 15px; }

.form-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 15px;
    background: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
}

.full-width { grid-column: span 2; }

.input-labeled { display: flex; flex-direction: column; text-align: left; }
.input-labeled label { font-size: 0.7rem; color: #7f8c8d; margin-bottom: 2px; padding-left: 5px; }

/* Estilo solicitado: fondo oscuro y texto blanco */
select, input {
    padding: 12px;
    border: 1px solid #ccc;
    border-radius: 4px;
    color: #fff;
    background: #2c3e50;
    transition: all 0.3s;
}

input:focus { border-color: #3498db; outline: none; }

.btn-success {
    grid-column: span 2;
    background: #27ae60;
    color: white;
    border: none;
    padding: 14px;
    cursor: pointer;
    border-radius: 4px;
    font-weight: bold;
}

.table-container {
    margin-top: 30px;
    background: white;
    padding: 15px;
    border-radius: 8px;
    color: #333;
}

table { width: 100%; border-collapse: collapse; }
th, td { padding: 14px; text-align: left; border-bottom: 1px solid #eee; }
th { background: #f9f9f9; text-transform: uppercase; font-size: 0.75rem; color: #7f8c8d; }

.td-actions { display: flex; gap: 8px; }

.status-ok { color: #27ae60; font-weight: bold; }
.status-err { color: #e74c3c; font-weight: bold; }

.btn-ok { background: #27ae60; color: white; border: none; padding: 8px 12px; border-radius: 4px; cursor: pointer; }
.btn-warn { background: #f39c12; color: white; border: none; padding: 8px 12px; border-radius: 4px; cursor: pointer; }
.btn-danger { background: #e74c3c; color: white; border: none; padding: 8px 12px; border-radius: 4px; cursor: pointer; }
</style>