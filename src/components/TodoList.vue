<script setup>
import { ref } from 'vue';

const props = defineProps(['state', 'username'])
let logged_in = ref(props.state)
let username = ref(props.username)
/*
 flags = enum {
    'local' means the item is stored on the user pc
    'removed' to be removed from db
    'db' means it's stored on the db
    'edited' really thinking about implimenting this later maybe even now actually
    'skip' doesn't matter
}
*/
let list = ref(JSON.parse(localStorage.getItem('list')) || [
    { label: "do something", state: true ,flag:'local'},
    { label: "learn backend", state: true ,flag:'local'}
])
let input = ref("")

function update() {
    localStorage.setItem('list', JSON.stringify(list.value));
    resetInactivityTimeout()
}
// on user open website -> user is not logged in by default
// when user login -> website refreshes
if (logged_in.value) {  // this should work
    pull()
}
function push() {
    fetch('http://localhost:3000/push', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        'list': list.value,
        'username': username.value
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
        .catch(error => {
            console.error('Error:', error);
        });
}
function pull() {
    console.log(username.value)
    fetch('http://localhost:3000/pull', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            'username': username.value
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
        .then(data => {
            // god please help me
            // gonna stop for now tomorrow will complete hopefully :)
            let newList = []
            data.list.forEach(element => {
                if
            });
            list.value = list.value + data.list;
        })
}
function onUserInactivity() {
    if (logged_in.value) {
        push()
    }
}
// Variable to store the timeout ID
let inactivityTimeout;

// Reset the inactivity timeout
function resetInactivityTimeout() {
    // Clear the existing timeout
    clearTimeout(inactivityTimeout);

    // Set a new timeout for 2 seconds
    inactivityTimeout = setTimeout(onUserInactivity, 1000);
}

resetInactivityTimeout();
function add() {
    if (input.value === "") {
        return;
    }
    list.value.push(
        {
            label: input.value,
            state: true
        }
    );
    input.value = "";
    update();
}
function remove(index) {
    list.value.splice(index, 1);
    update();

}
function done(index) {
    list.value[index].state = !list.value[index].state;
    update();

}
</script>

<template>
    <div>
        <input type="text" v-model="input">
        <button @click="add">Add</button>
        <ul>
            <li v-for="(element, index) in list" :key="index">

                <input type="checkbox" v-on:change="done(index)">

                <span v-if="element.state">{{ element.label }}</span>
                <span v-else style="text-decoration: line-through;">{{ element.label }}</span>

                <span @click="remove(index)" style="cursor: pointer;">❌</span>

            </li>
        </ul>
        <a v-if="logged_in === false" href="#/login"><button>Log In</button></a>
        <a v-if="logged_in === true" v-on:click="logged_in = false"> <button>Log Out</button></a>
    </div>
</template>
