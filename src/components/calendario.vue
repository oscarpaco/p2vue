<script setup>
import { ref, computed } from 'vue'

// 1. Generamos los próximos 7 días de forma automática
const obtenerProximosDias = () => {
  const dias = []
  const opciones = { weekday: 'short', day: 'numeric' }
  
  for (let i = 0; i < 12; i++) {
    const fecha = new Date()
    fecha.setDate(fecha.getDate() + i)
    // Guardamos una versión fácil de leer (ej: "2026-07-09") y una bonita para la pantalla (ej: "jue 9")
    dias.push({
      id: fecha.toISOString().split('T')[0],
      nombreCorto: fecha.toLocaleDateString('es-ES', opciones)
    })
  }
  return dias
}

const proximosDias = obtenerProximosDias()

// 2. VARIABLES REACTIVAS
const diaSeleccionado = ref(proximosDias[0].id) // Empieza con el día de hoy seleccionado
const nuevaTarea = ref('') // Lo que el usuario escribe en el input

// Diccionario reactivo para guardar las tareas de cada día
// Ejemplo: { '2026-07-09': ['Comprar leche', 'Ir al gym'], '2026-07-10': [] }
const tareasPorDia = ref({})

// 3. VARIABLE COMPUTADA (Reactiva)
// Obtiene automáticamente solo las tareas del día que está haciendo click el usuario
const tareasDelDiaActual = computed(() => {
  return tareasPorDia.value[diaSeleccionado.value] || []
})

// 4. FUNCIONES (Acciones)
const agregarTarea = () => {
  if (nuevaTarea.value.trim() === '') return // Si está vacío, no hace nada

  // Si el día seleccionado aún no tiene tareas, le creamos una lista vacía
  if (!tareasPorDia.value[diaSeleccionado.value]) {
    tareasPorDia.value[diaSeleccionado.value] = []
  }

  // Agregamos la nueva tarea a ese día
  tareasPorDia.value[diaSeleccionado.value].push(nuevaTarea.value)
  
  // Limpiamos el input reactivo
  nuevaTarea.value = ''
}
</script>

<template>
  <div class="calendario-contenedor">
    <h2>📅 Agenda Semanal</h2>

    <!-- LISTA DE DÍAS (Botones) -->
    <div class="dias-lista">
      <button 
        v-for="dia in proximosDias" 
        :key="dia.id"
        :class="{ activo: diaSeleccionado === dia.id }"
        @click="diaSeleccionado = dia.id"
      >
        {{ dia.nombreCorto }}
        <!-- Un puntito si el día ya tiene tareas registradas -->
        <span v-if="tareasPorDia[dia.id]?.length > 0" class="punto-tarea">.</span>
      </button>
    </div>

    <!-- SECCIÓN DE TAREAS DEL DÍA -->
    <div class="tareas-seccion">
      <h3>Tareas para el {{ proximosDias.find(d => d.id === diaSeleccionado).nombreCorto }}</h3>

      <!-- Formulario para añadir tarea -->
      <form @submit.prevent="agregarTarea" class="formulario">
        <input 
          v-model="nuevaTarea" 
          type="text" 
          placeholder="Escribe una nueva tarea..."
        >
        <button type="submit">Añadir</button>
      </form>

      <!-- Lista de tareas en pantalla -->
      <ul v-if="tareasDelDiaActual.length > 0">
        <li v-for="(tarea, index) in tareasDelDiaActual" :key="index">
          ✅ {{ tarea }}
        </li>
      </ul>
      <p v-else class="sin-tareas">No tienes tareas para este día. ¡A descansar!</p>
    </div>
  </div>
</template>

<style scoped>
/* Un poco de estilo para que se vea genial */
.calendario-contenedor {
  font-family: sans-serif;
  max-width: 500px;
  margin: 20px auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}
.dias-lista {
  display: flex;
  gap: 8px;
  overflow-x: auto;
  padding-bottom: 10px;
  margin-bottom: 20px;
}
.dias-lista button {
  padding: 10px;
  border: 1px solid #ccc;
  background: white;
  border-radius: 8px;
  cursor: pointer;
  position: relative;
  min-width: 60px;
}
.dias-lista button.activo {
  background: #42b883; /* El verde clásico de Vue */
  color: white;
  border-color: #42b883;
}
.punto-tarea {
  position: absolute;
  bottom: 2px;
  left: 50%;
  transform: translateX(-50%);
  color: red;
  font-weight: bold;
  font-size: 16px;
}
.formulario {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}
.formulario input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.formulario button {
  background: #35495e;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 4px;
  cursor: pointer;
}
ul { list-style: none; padding: 0; }
li { padding: 8px; background: #f9f9f9; margin-bottom: 5px; border-radius: 4px; }
.sin-tareas { color: #666; font-style: italic; }
</style>