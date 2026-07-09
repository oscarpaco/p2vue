<script setup>
import { ref, computed } from 'vue'

// 1. El precio base del producto (fijo)
const precioUnitario = 100

// 2. Variable reactiva: la cantidad de productos (empieza en 1)
const cantidad = ref(1)

// 3. Variable reactiva: si el cupón está activado o no (empieza en falso)
const cuponAplicado = ref(false)

// 4. Variable reactiva calculada: el total de la compra
const total = computed(() => {
  let subtotal = precioUnitario * cantidad.value
  
  // Si la variable del cupón es true, restamos el 10%
  if (cuponAplicado.value) {
    subtotal = subtotal * 0.9
  }
  
  return subtotal
})
</script>

<template>
  <div class="carrito">
    <h2>Tu Carrito</h2>
    <p>Producto: Camiseta Premium — <strong>${{ precioUnitario }}</strong></p>

    <div class="controles">
      <button @click="cantidad.value > 1 ? cantidad.value-- : null">-</button>
      <span> Cantidad: {{ cantidad }} </span>
      <button @click="cantidad.value++">+</button>
    </div>

    <div class="cupon">
      <label>
        <input type="checkbox" v-model="cuponAplicado">
        ¿Tienes el cupón de descuento "VUE10"?
      </label>
    </div>

    <hr>

    <h3>Total a pagar: ${{ total }}</h3>
  </div>
</template>