<template>
    <div>
        <h1>Value : {{ inputValue }}</h1>
        <input type="text" v-bind:value="inputValue">
        <button @click="handleClick">Click</button>
    </div>
</template>

<script setup>
    import { ref } from 'vue';
    const inputValue = ref("Anh");
    const handleClick = () =>{
        inputValue.value = "Quang";
    }
</script>

<style scoped>

</style>