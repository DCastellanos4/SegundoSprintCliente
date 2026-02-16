<script setup>
/**
 * COMPONENTE: GestionEtapas.vue
 * Descripción: Mantenimiento de las etapas educativas del centro.
 * Permite registrar, editar y eliminar con validación de integridad.
 */
import { ref, onMounted } from 'vue'

// --- ESTADO REACTIVO ---
const listaEtapas = ref([])
const etapaEnEdicion = ref(null)

const modeloEtapa = ref({ id: '', nombre: '', descripcion: '' })

/**
 * Carga las etapas desde la API y las ordena por ID.
 */
const sincronizarEtapas = async () => {
    try {
        const respuesta = await fetch('http://100.52.46.68:3000/etapas')
        const datos = await respuesta.json()

        // Ordenamos por ID numérico para que la tabla sea estable
        listaEtapas.value = datos.sort((a, b) => Number(a.id) - Number(b.id))
    } catch (error) {
        console.error("Fallo al sincronizar etapas:", error);
    }
}

/**
 * Procesa el formulario. Decide si crear (POST) o actualizar (PUT).
 */
const procesarFormulario = async () => {
    // Si estamos editando, usamos los datos de 'etapaEnEdicion', si no, de 'modeloEtapa'
    const datosParaEnviar = etapaEnEdicion.value ? etapaEnEdicion.value : modeloEtapa.value;
    const metodo = etapaEnEdicion.value ? 'PUT' : 'POST';
    const url = etapaEnEdicion.value
        ? `http://100.52.46.68:3000/etapas/${etapaEnEdicion.value.id}`
        : 'http://100.52.46.68:3000/etapas';

        //ERROR BBDD:
    // Si es un registro nuevo, calculamos el ID autoincremental manual
    if (!etapaEnEdicion.value) {
        const maxId = listaEtapas.value.length > 0
            ? Math.max(...listaEtapas.value.map(e => Number(e.id)))
            : 0;
        datosParaEnviar.id = (maxId + 1).toString();
    }

    try {
        const respuesta = await fetch(url, {
            method: metodo,
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(datosParaEnviar)
        });

        if (respuesta.ok) {
            alert(etapaEnEdicion.value ? "Etapa actualizada" : "Etapa registrada con éxito");
            cancelarAccion();
            await sincronizarEtapas();
        }
    } catch (error) {
        alert("Error de comunicación con el servidor.");
    }
}

/**
 * Elimina una etapa si el usuario confirma.
 */
const eliminarRegistro = async (id) => {
    const confirmacion = confirm("¿Seguro que quieres borrar esta etapa? Se perderán sus vínculos.");

    if (confirmacion) {
        try {
            const respuesta = await fetch(`http://100.52.46.68:3000/etapas/${id}`, { method: 'DELETE' });

            if (respuesta.ok) {
                await sincronizarEtapas();
            } else {
                alert("Error: No se puede borrar (posiblemente tiene cursos asociados).");
            }
        } catch (error) {
            console.error("Fallo en el borrado:", error);
        }
    }
}

// --- GESTIÓN DE UI ---
const prepararEdicion = (etapa) => { etapaEnEdicion.value = { ...etapa }; }
const cancelarAccion = () => {
    etapaEnEdicion.value = null;
    modeloEtapa.value = { id: '', nombre: '', descripcion: '' };
}

onMounted(sincronizarEtapas);
</script>

<template>
    <div class="gestion-container">
        <header class="seccion-header">
            <h3>Gestión de Etapas Educativas (H8)</h3>
        </header>

        <form @submit.prevent="procesarFormulario" :class="['form-grid', { 'modo-edicion': etapaEnEdicion }]">

            <h4 v-if="etapaEnEdicion">Editando Etapa: {{ etapaEnEdicion.nombre }}</h4>

            <template v-if="!etapaEnEdicion">
                <input v-model="modeloEtapa.nombre" placeholder="Nombre (ej. Grado Superior)" required>
                <input v-model="modeloEtapa.descripcion" placeholder="Descripción breve..." required>
            </template>

            <template v-else>
                <input v-model="etapaEnEdicion.nombre" placeholder="Nombre" required>
                <input v-model="etapaEnEdicion.descripcion" placeholder="Descripción" required>
            </template>

            <div class="form-actions">
                <button type="submit" class="btn-success">
                    {{ etapaEnEdicion ? 'Actualizar Datos' : 'Registrar Etapa' }}
                </button>
                <button v-if="etapaEnEdicion" type="button" @click="cancelarAccion" class="btn-sec">
                    Cancelar
                </button>
            </div>
        </form>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Etapa</th>
                        <th>Descripción</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="et in listaEtapas" :key="et.id">
                        <td><strong>#{{ et.id }}</strong></td>
                        <td>{{ et.nombre }}</td>
                        <td>{{ et.descripcion || 'Sin descripción' }}</td>
                        <td class="action-buttons">
                            <button @click="prepararEdicion(et)" class="btn-accent">Editar</button>
                            <button @click="eliminarRegistro(et.id)" class="btn-danger">Borrar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<style scoped>
.seccion-header {
    margin-bottom: 20px;
    border-left: 5px solid #27ae60;
    padding-left: 15px;
}

.form-grid {
    display: grid;
    gap: 15px;
    background: #fff;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 25px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.modo-edicion {
    border: 2px solid #f39c12;
    background: #fffcf5;
}

.form-actions {
    display: flex;
    gap: 10px;
}

/* Inputs estilo oscuro DAW */
input {
    padding: 12px;
    border: 1px solid #ccc;
    border-radius: 6px;
    color: #fff;
    background: #2c3e50;
    transition: all 0.3s;
}

input:focus {
    border-color: #3498db;
    outline: none;
}

.btn-success {
    background: #27ae60;
    color: white;
    border: none;
    padding: 12px;
    cursor: pointer;
    border-radius: 4px;
    font-weight: bold;
    flex: 2;
}

.btn-sec {
    background: #95a5a6;
    color: white;
    border: none;
    padding: 12px;
    cursor: pointer;
    border-radius: 4px;
    flex: 1;
}

.btn-accent {
    background: #f39c12;
    color: white;
    border: none;
    padding: 8px 15px;
    border-radius: 4px;
    cursor: pointer;
}

.btn-danger {
    background: #e74c3c;
    color: white;
    border: none;
    padding: 8px 15px;
    border-radius: 4px;
    cursor: pointer;
}

.action-buttons {
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
    padding: 14px;
    text-align: left;
    border-bottom: 1px solid #f1f1f1;
}

th {
    background: #f9f9f9;
    text-transform: uppercase;
    font-size: 0.8rem;
    color: #7f8c8d;
}
</style>