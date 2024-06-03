<script setup>

const emit = defineEmits(['test']);
function handelsubmit  ()  {
    window.location.hash = "#/"
    emit('test')
}
let username = ''
let password = ''
</script>

<template>
    <div id="container">
            <label for="username">Username: </label>
            <input type="text" v-model="username">
            <label for="password">Password: </label>
            <input type="password" v-model="password">
            <button type="submit" @click="handelsubmit">Login</button>
    </div>
</template>

<style>
#container {
    display: grid;
    justify-content: center;
    align-items: center;
}
</style>
