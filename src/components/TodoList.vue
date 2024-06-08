<script setup>
import { ref } from 'vue';

const props = defineProps(['state', 'username'])
let logged_in = ref(props.state)
let username = ref(props.username)
/*
 flags = enum {
    'added' newly added so stored locally
    'removed' to be removed from db
    'db' means it's stored on the db
    'edited' really thinking about implimenting this later maybe even now actually
    'skip' doesn't matter
}
    🤓🤓🤓🤓🤓🤓
*/

let list = ref(JSON.parse(localStorage.getItem('list')) || [
    { label: "do something", state: true, flag: 'local' },
    { label: "learn backend", state: true, flag: 'local' }
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
    let new_list = []
    for (let i = 0; i < list.value.length; i++) {
        const element = list.value[i];
        if (element.flag !== 'db') {
            new_list.push(element)
        }
        if (element.flag === 'removed') {
            list.value.splice(i, 1)
        }
    }
    fetch('http://localhost:3000/push', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            'list': new_list,
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
    list.value.forEach(element => {
        element.flag = 'db'
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

            data.list.forEach(element => {
                let index = list.value.findIndex((l) => l.label === element.label)
                console.log(index)
                if (index === -1) {
                    list.value.push({
                        label: element.label,
                        state: element.state,
                        flag: 'db'
                    })
                }
                else {
                    list.value[index].state = element.state
                    list.value[index].flag = 'db'
                }

            });
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
            state: true,
            flag: 'added'
        }
    );
    input.value = "";
    update();
}
function remove(index) {
    list.value[index].flag = 'removed'
    //list.value.splice(index, 1);
    update();

}
function done(index) {
    list.value[index].state = !list.value[index].state;
    list.value[index].flag = 'edited'
    update();

}
</script>

<template>
    <div>
        <input type="text" v-model="input">
        <button @click="add">Add</button>
        <ul>
            <span  v-for="(element, index) in list" :key="index" >
            <li  v-if="element.flag != 'removed'">
                    <input type="checkbox" v-on:change="done(index)">

                    <span v-if="element.state">{{ element.label }}</span>
                    <span v-else style="text-decoration: line-through;">{{ element.label }}</span>

                    <span @click="remove(index)" style="cursor: pointer;">❌</span>
                </li>
            </span>
        </ul>
        <a v-if="logged_in === false" href="#/login"><button>Log In</button></a>
        <a v-if="logged_in === true" v-on:click="logged_in = false"> <button>Log Out</button></a>
    </div>
</template>
