<script setup>
import { ref } from 'vue';

const props = defineProps(['state', 'username'])
let logged_in = ref(props.state)
let username = ref(props.username)


let list = ref(JSON.parse(localStorage.getItem('list')) || [
    { label: "do something", state: true },
    { label: "learn backend", state: true }
])
let edits = ref([])
let input = ref("")

function update() {
    localStorage.setItem('list', JSON.stringify(list.value));
    resetInactivityTimeout()
}
// on user open website -> user is not logged in by default
// when user login -> website refreshes
if (logged_in.value) {  // this should work
    let old = list.value;
    pull()
    HardPush(old)
}
function HardPush(list) {
    fetch('http://localhost:3000/hardpush', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            'list': list,
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
function push() {

    fetch('http://localhost:3000/push', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            'list': edits.value,
            'username': username.value
        })
    })

        .then(response => {
            // Check if the request was successful
            if (!response.ok) {
                throw new Error('Network response was not ok');
            }
            // Parse the response as JSON
            edits.value = []
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

            data.list.forEach(element => {
                let index = list.value.findIndex((l) => l.label === element.label)
                console.log(index)
                if (index === -1) {
                    list.value.push({
                        label: element.label,
                        state: element.state,
                    })
                }
                else {
                    list.value[index].state = element.state
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
        }
    );
    edits.value.push({
        label: input.value,
        type: 'add',
        info: {
            state: true
        }
    })
    input.value = "";
    update();
}
function remove(index) {
    edits.value.push({
        label: list.value[index].label,
        type: 'remove',
        info: {}
    })
    list.value.splice(index, 1)
    update()

}
function done(index) {
    list.value[index].state = !list.value[index].state;
    edits.value.push({
        label: list.value[index].label,
        type: 'edit',
        info: {
            collum: 'state',
            value: list.value[index].state
        }
    })
    update();

}
</script>

<template>
    <div>
        <input type="text" v-model="input">
        <button @click="add">Add</button>
        <ul>
            <span v-for="(element, index) in list" :key="index">
                <li>
                    <input type="checkbox" v-on:click="done(index)" v-model="element.state">
                    <span v-if="element.state"  style="text-decoration: line-through;">{{ element.label }}</span>
                    <span v-else >{{ element.label }}</span>
                    <span @click="remove(index)" style="cursor: pointer;">❌</span>
                </li>
            </span>
        </ul>
        <a v-if="logged_in === false" href="#/login"><button>Log In</button></a>
        <a v-if="logged_in === true" v-on:click="logged_in = false"> <button>Log Out</button></a>
    </div>
</template>
