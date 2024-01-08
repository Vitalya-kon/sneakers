<script setup>
import { reactive, watch, ref, onMounted } from 'vue'
import axios from 'axios'
import debounce from 'lodash.debounce'
import { inject } from 'vue'
import CardList from '../components/CardList.vue'

const { cart, addToCart, removeFromCart } = inject('cart')

const items = ref([])

const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

const onChangeSelect = (e) => {
  filters.sortBy = e.target.value
}
const onChangeSearchInput = debounce((e) => {
  filters.searchQuery = e.target.value
}, 500)

const onClickAddPluse = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        item_id: item.id
      }

      item.isFavorite = true

      const { data } = await axios.post(`https://32dfb09ec9b9beb4.mokky.dev/favorites`, obj)
      item.favoriteId = data.id

    } else {
      item.isFavorite = false
      await axios.delete(`https://32dfb09ec9b9beb4.mokky.dev/favorites/${item.favoriteId}`)

      item.favoriteId = null
    }
  } catch (error) {
    console.log(error)
  }
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://32dfb09ec9b9beb4.mokky.dev/favorites`)

    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.item_id === item.id)

      if (!favorite) {
        return item
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (error) {
    console.log(error)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }

    const { data } = await axios.get(`https://32dfb09ec9b9beb4.mokky.dev/items`, { params })

    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    }))
  } catch (error) {
    console.log(error)
  }
}

onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localStorage ? JSON.parse(localCart) : []

  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded:false
  }))
})

watch(filters, fetchItems)
</script>
<template>
  <div>
    <div class="flex justify-between items-center">
      <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

      <div class="flex gap-4">
        <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
          <option value="name">По названию</option>
          <option value="price">По цене (дешевле)</option>
          <option value="-price">По цене (дороже)</option>
        </select>

        <div class="relative">
          <img src="/search.svg" class="absolute left-4 top-3" alt="Search" />
          <input
            @input="onChangeSearchInput"
            class="border border-slate-200 pl-11 pr-4 outline-none py-2 rounded-md focus:border-slate-400"
            type="text"
            placeholder="Поиск.."
          />
        </div>
      </div>
    </div>
    <div class="mt-10">
      <CardList :items="items" @addToFavorite="addToFavorite" @add-to-cart="onClickAddPluse" />
    </div>
  </div>
</template>