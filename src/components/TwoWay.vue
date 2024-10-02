<template>
    <div>
        <input type="text" v-model="inputValue">
        <p>Giá trị input value : {{ inputValue }}</p>
        <button @click="handleClick">Click</button>
        <p>**********</p>
        <ul>
            <p>Type = checkbox</p>
            <li><input type="checkbox" v-model="hobbies" value="đá bóng">đá bóng</li>
            <li><input type="checkbox" v-model="hobbies" value="cầu lông">cầu lông</li>
            <li><input type="checkbox" v-model="hobbies" value="gym">gym</li>
        </ul>
        <p>Sở thích đã được chọn {{ hobbies }}</p>
        <p>**********</p>
        <ul>
            <p>Type = radio</p>
            <li><input type="radio" value="nam" v-model="gender"><label for="nam">nam</label></li>
            <li><input type="radio" value="nữ" v-model="gender"><label for="nữ">nữ</label></li>
            <li><input type="radio" value="khác" v-model="gender"><label for="khác">khác</label></li>
        </ul>
        <p>Giới tính : {{ gender }}</p>
    </div>
</template>

<script setup>
    import { ref } from 'vue';
    const inputValue = ref("Anh");
    const handleClick = () =>{
        inputValue.value = "Quang";
    }
    const hobbies = ref([]);
    const gender = ref("nam");
</script>

<style scoped>

</style>