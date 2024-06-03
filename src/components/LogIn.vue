<script setup>
let username = ''
let password = ''

const emit = defineEmits(['test']);
function handelsubmit() {
    fetch('http://localhost:3000/login', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        'username': username,
        'password': password
    })
})
.then(response => {
    // Check if the request was successful
    if (!response.ok) {
        throw new Error('Network response was not ok');
    }
    // Parse the response as JSON
    return response.json();
})
.then(response => {
    if (response.state) {
        // Assuming emit is a valid function in your context
        emit('test');
        window.location.hash = "#/";
    } else {
        console.log("user not found");
    }
})
.catch(error => {
    console.error('Error:', error);
});
}
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
