<script setup>
import { ref, inject, computed } from 'vue'
import axios from 'axios'

import DrawerHead from './DrawerHead.vue'
import CartListItem from './CartListItem.vue'
import InfoBlock from './InfoBlock.vue'


const props = defineProps({
  totalPrice: Number,
  vatPrice: Number,
})

const { cart } = inject('cart')

const isCreating = ref(false)
const orderId = ref(null)
const createOrder = async () => {
  try {
    isCreating.value = true

    const { data } = await axios.post(`https://32dfb09ec9b9beb4.mokky.dev/orders`, {
      items: cart.value,
      totalPrice: props.totalPrice.value
    })

    cart.value = []

    orderId.value = data.id;
  } catch (error) {
    console.log(error)
  } finally {
    isCreating.value = false
  }
}

const cartIsEmpty = computed(() => {
  return cart.value.length === 0
})

const buttonDisabled = computed(() => {
  return isCreating.value || cartIsEmpty.value
})
</script>

<template>
  <div>
    <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>
    <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
      <DrawerHead />
      
      <div v-if="!totalPrice || orderId" class="flex h-full items-center">
        <InfoBlock
          v-if="!totalPrice && !orderId"
          title="Корзина пустая"
          description="Добавьте хотя бы одну пару кроссовок, чтобы сделать заказ."
          image-url="/package-icon.png"
        />
        <InfoBlock
          v-if="orderId"
          title="Заказ оформлен"
          :description="`Ваш заказ #${orderId} начали готовить, ожидайте звонка менеджера`"
          image-url="/order-success-icon.png"
        />
      </div>
      <div v-else>
        <CartListItem />

        <div class="flex flex-col gap-4 mt-7">
          <div class="flex gap-2">
            <span>Итого:</span>
            <div class="flex-1 border-b border-dashed"></div>
            <b>{{ totalPrice }} Р</b>
          </div>

          <div class="flex gap-2">
            <span>Налог 5%:</span>
            <div class="flex-1 border-b border-dashed"></div>
            <b>{{ vatPrice }} Р</b>
          </div>
          <button
            :disabled="buttonDisabled"
            @click="createOrder"
            class="bg-lime-500 text-white p-2 w-full rounded-xl py-3 hover:bg-lime-600 active:bg-lime-700 transition disabled:bg-slate-300 cursor-pointer mt-4"
          >
            Оформить заказ
          </button>
        </div>
      </div>
    </div>
  </div>
</template>