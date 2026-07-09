<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import L from 'leaflet'

// 1. VARIABLES REACTIVAS
const latitud = ref(40.4167)
const longitud = ref(-3.7037)

const mapaContenedor = ref(null)
let mapaInstance = null
let marcadorInstance = null
let circuloInstance = null // Para guardar la referencia del círculo activo
let resizeObserver = null   // Para vigilar el tamaño del mapa

// Solución para que los iconos de Leaflet no se rompan
const corregirIconoDefecto = () => {
  delete L.Icon.Default.prototype._getIconUrl
  L.Icon.Default.mergeOptions({
    iconRetinaUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-icon-2x.png',
    iconUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-icon.png',
    shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
  })
}

// 2. CONFIGURACIÓN AL MONTAR EL COMPONENTE
onMounted(() => {
  corregirIconoDefecto()

  // Crear el mapa base
  mapaInstance = L.map(mapaContenedor.value).setView([latitud.value, longitud.value], 16)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap contributors'
  }).addTo(mapaInstance)

  // Crear el marcador inicial (arrastrable)
  marcadorInstance = L.marker([latitud.value, longitud.value], { draggable: true }).addTo(mapaInstance)

  // Si arrastramos el marcador, actualizamos los inputs
  marcadorInstance.on('dragend', () => {
    const posicion = marcadorInstance.getLatLng()
    latitud.value = parseFloat(posicion.lat.toFixed(4))
    longitud.value = parseFloat(posicion.lng.toFixed(4))
    
    // Si ya existe un círculo, lo movemos también junto al marcador
    if (circuloInstance) {
      circuloInstance.setLatLng(posicion)
    }
  })

  // 🔥 LA SOLUCIÓN DEFINITIVA AL TABLERO DE AJEDREZ:
  // Creamos un observador que detecta cuándo el DIV del mapa cambia de tamaño real en la pantalla
  resizeObserver = new ResizeObserver(() => {
    if (mapaInstance) {
      mapaInstance.invalidateSize()
    }
  })
  
  // Empezamos a vigilar el div del mapa
  if (mapaContenedor.value) {
    resizeObserver.observe(mapaContenedor.value)
  }
})

// Limpieza al destruir el componente para evitar fugas de memoria
onBeforeUnmount(() => {
  if (resizeObserver) {
    resizeObserver.disconnect()
  }
})

// 3. ACCIÓN: UBICAR PUNTO Y DIBUJAR CÍRCULO
const ubicarPuntoManual = () => {
  const lat = parseFloat(latitud.value)
  const lng = parseFloat(longitud.value)

  if (!isNaN(lat) && !isNaN(lng) && mapaInstance && marcadorInstance) {
    const nuevasCoordenadas = [lat, lng]

    // 1. Mover marcador y mapa
    marcadorInstance.setLatLng(nuevasCoordenadas)
    mapaInstance.setView(nuevasCoordenadas, 16)

    // 2. Borrar el círculo anterior si ya existía uno
    if (circuloInstance) {
      mapaInstance.removeLayer(circuloInstance)
    }

    // 3. Dibujar el nuevo círculo pequeño (radius está en metros)
    circuloInstance = L.circle(nuevasCoordenadas, {
      color: '#ff4757',       // Color del borde (Rojo)
      fillColor: '#ff6b81',   // Color del relleno
      fillOpacity: 0.4,       // Transparencia del relleno
      radius: 50              // Radio pequeño (50 metros a la redonda)
    }).addTo(mapaInstance)

    // Forzar redibujado de seguridad
    mapaInstance.invalidateSize()
  } else {
    alert("Por favor, introduce coordenadas válidas.")
  }
}
</script>

<template>
  <div class="mapa-componente">
    <h2>📍 Ubicar Punto en el Mapa</h2>

    <div class="formulario-contenedor">
      <div class="inputs-contenedor">
        <div class="campo">
          <label>Eje X (Latitud):</label>
          <input type="number" v-model="latitud" step="any"> 
        </div>
        
        <div class="campo">
          <label>Eje Y (Longitud):</label>
          <input type="number" v-model="longitud" step="any">
        </div>
      </div>

      <button type="button" class="btn-ubicar" @click="ubicarPuntoManual">
        🔍 Ubicar Punto
      </button>
    </div>

    <div ref="mapaContenedor" class="mi-mapa"></div>
  </div>
</template>

<style scoped>
@import "leaflet/dist/leaflet.css";

.mapa-componente {
  font-family: sans-serif;
  max-width: 650px;
  margin: 20px auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.05);
  background: #ffffff;
}

.formulario-contenedor {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 20px;
}

.inputs-contenedor {
  display: flex;
  gap: 15px;
}

.campo {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.campo label {
  font-weight: bold;
  margin-bottom: 5px;
  color: #333;
  font-size: 14px;
}

.campo input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 16px;
  outline: none;
}

.campo input:focus {
  border-color: #42b883;
}

.btn-ubicar {
  background-color: #42b883;
  color: white;
  border: none;
  padding: 12px;
  font-size: 16px;
  font-weight: bold;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.2s ease;
}

.btn-ubicar:hover {
  background-color: #35495e;
}

.mi-mapa {
  height: 400px;
  width: 100%;
  border-radius: 8px;
  border: 1px solid #ccc;
  position: relative;
  overflow: hidden !important;
}
</style>