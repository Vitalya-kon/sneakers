<script setup>
import {  provide, ref, watch, computed } from 'vue'

import Header from './components/Header.vue'
import Drawer from './components/Drawer.vue'


// КОРЗИНА
const cart = ref([])
const drawerOpen = ref(false)

const totalPrice = computed(() => {
  return cart.value.reduce((acc, item) => acc + item.price, 0)
})
const vatPrice = computed(() => {
  return Math.round((totalPrice.value * 5) / 100)
})


const closeDrawer = () => {
  drawerOpen.value = false
}
const openDrawer = () => {
  drawerOpen.value = true
}



const addToCart = (item) => {
  cart.value.push(item)
  item.isAdded = true
}

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1)
  item.isAdded = false
}



watch(cart, () => {
  localStorage.setItem('cart', JSON.stringify(cart.value))
}, {
  deep: true
})

provide('cart', {
  cart,
  closeDrawer,
  openDrawer,
  addToCart,
  removeFromCart
})

// КОРЗИНА END

</script>

<template>
  <div class="w-4/5 m-auto bg-white rounded-xl shadow-xl mt-14">
    <Drawer
      v-if="drawerOpen"
      :total-price="totalPrice"
      :vatPrice="vatPrice"
      
    />
    <Header :total-price="totalPrice" @openDrawer="openDrawer" />

    <div class="p-10">
      <router-view></router-view>
    </div>
  </div>
</template>

