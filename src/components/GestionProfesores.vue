<script setup>
import { ref, onMounted } from 'vue'

const profesores = ref([])
const departamentos = ref([])
const roles = ref([])

const nuevoProfesor = ref({
    dni_nie: '',
    nombre: '',
    apellidos: '',
    correo_institucional: '',
    departamento_id: '',
    rol_id: 2,
    password_hash: ''
})

const cargarDatosIniciales = async () => {
    try {
        const [resProf, resDep, resRoles] = await Promise.all([
            fetch('http://100.52.46.68:3000/profesores'),
            fetch('http://100.52.46.68:3000/departamentos'),
            fetch('http://100.52.46.68:3000/roles')
        ]);
        profesores.value = await resProf.json();
        departamentos.value = await resDep.json();
        roles.value = await resRoles.json();
    } catch (error) {
        console.error("Error cargando tablas:", error);
    }
}

const guardarProfesor = async () => {
    // CA1: Validación de duplicados
    const duplicado = profesores.value.find(p =>
        p.dni_nie === nuevoProfesor.value.dni_nie ||
        p.correo_institucional === nuevoProfesor.value.correo_institucional
    );
    if (duplicado) {
        alert("Error: El DNI o el Correo ya existen");
        return;
    }
    await fetch('http://100.52.46.68:3000/profesores', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(nuevoProfesor.value)
    });
    nuevoProfesor.value = { dni_nie: '', nombre: '', apellidos: '', correo_institucional: '', departamento_id: '', rol_id: '', password_hash: '' };
    cargarDatosIniciales();
}

// NUEVA FUNCIÓN: Borrado directo sin modal
const borrarDirecto = async (id) => {
    if (!confirm("¿Seguro que quieres eliminar este profesor?")) return;

    try {
        // H3: Realizamos el borrado directo. 
        // Nota: Si la API exige el parámetro reservas, añadimos ?reservas=cancelar por defecto.
        const url = `http://100.52.46.68:3000/profesores/${id}`;

        const respuesta = await fetch(url, { method: 'DELETE' });

        if (respuesta.ok) {
            alert("Profesor eliminado");
            await cargarDatosIniciales(); // Recargamos la lista
        } else {
            const txt = await respuesta.text();
            alert("Error del servidor: " + txt);
        }
    } catch (error) {
        console.error("Error de red:", error);
        alert("No se pudo conectar con el servidor.");
    }
};

onMounted(cargarDatosIniciales);
</script>

<template>
    <div class="gestion-container">
        <h3>Gestión de Profesores (H3)</h3>

        <form @submit.prevent="guardarProfesor" class="form-grid">
            <input v-model="nuevoProfesor.dni_nie" placeholder="DNI/NIE" required>
            <input v-model="nuevoProfesor.nombre" placeholder="Nombre" required>
            <input v-model="nuevoProfesor.apellidos" placeholder="Apellidos" required>
            <input v-model="nuevoProfesor.correo_institucional" type="text" placeholder="Correo Institucional" required>

            <select v-model="nuevoProfesor.departamento_id" required>
                <option value="" disabled>Seleccione Departamento</option>
                <option v-for="d in departamentos" :key="d.id" :value="d.id">{{ d.nombre }}</option>
            </select>

            <input v-model="nuevoProfesor.password_hash" type="text" placeholder="Contraseña" required>
            <button type="submit" class="btn-success">Registrar Profesor</button>
        </form>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>Profesor</th>
                        <th>DNI</th>
                        <th>Departamento</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="p in profesores" :key="p.id">
                        <td>{{ p.nombre }} {{ p.apellidos }}</td>
                        <td>{{ p.dni_nie }}</td>
                        <td>{{departamentos.find(d => d.id == p.departamento_id)?.nombre || 'N/A'}}</td>
                        <td>
                            <button @click="borrarDirecto(p.dni_nie)" class="btn-danger">Eliminar</button>
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

select,
input {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    color: #fff;
}

input::placeholder {
    color: grey;
}

.btn-success {
    grid-column: span 2;
    background: #27ae60;
    color: white;
    border: none;
    padding: 12px;
    cursor: pointer;
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

.btn-danger {
    background: #e74c3c;
    color: white;
    padding: 8px;
    cursor: pointer;
    border: none;
    border-radius: 4px;
}
</style>