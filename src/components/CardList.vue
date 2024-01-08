<script setup>

    import Card from './Card.vue'

    defineProps({
        items: Array,
        isFavorites: Boolean
    })

    const emit = defineEmits(['addToFavorite', 'addToCart']);
    

</script>

<template>
    <div class="grid grid-cols-4 gap-5 " v-auto-animate>
        <Card 
            v-for="item in items"
            :key="item.id"
            :title="item.title" 
            :image-url='item.imageUrl' 
            :price="item.price"
            :onClickFavorite="isFavorites ? null : () => emit('addToFavorite', item)"
            :onClickAdd="isFavorites ? null : () => emit('addToCart', item)"
            :isFavorite="item.isFavorite"
            :isAdded="item.isAdded"
            :id="item.id"
        />
    </div>
</template>