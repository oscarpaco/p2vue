<template>
<div class="buscador">
    <input type="text" v-model="busqueda" @keyup.enter="buscar" placeholder="Buscar...">
    <p>Buscando: {{ busqueda }}</p>
    <ul>
        <li v-for="amigo in amigosFiltrados" :key="amigo">
            {{ amigo.nombre }}
            </li>
    </ul>

</div>
</template>
<script setup>
import { ref, computed } from 'vue'
const busqueda = ref('');
const amigos = ref([
    { nombre: 'Juan' },
    { nombre: 'María' },
    { nombre: 'Pedro' },
    { nombre: 'Ana' },
    { nombre: 'Luis' }
])
const amigosFiltrados = computed(() => { 
    return amigos.value.filter(amigo => 
        amigo.nombre.toLowerCase().includes(busqueda.value.toLowerCase())
    )
})
</script>
