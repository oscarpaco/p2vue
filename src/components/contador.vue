<template>
 <h1>hola mundo</h1> <br><br><br>
{{ contador }} <br><br><br>
<button @click="incrementar">Incrementar</button>
<button @click="decrementar">Decrementar</button>
</template>

<script setup>
import { ref } from 'vue'
const contador = ref(0);
const incrementar = () => {
  contador.value++;
}
const decrementar = () => {
  contador.value--;
}
</script>

<style scoped>

</style>
