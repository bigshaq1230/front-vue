<script setup>
import { ref } from 'vue';

const props = defineProps(['state'])
let logged_in = ref(props.state)
let list = ref(JSON.parse(localStorage.getItem('list')) || [
    { label: "do something", state: true },
    { label: "learn backend", state: true }
])
let input = ref("")
function update() {
    localStorage.setItem('list', JSON.stringify(list.value));
}
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
        <a v-if="logged_in === false" href="#/login" ><button>Log In</button></a>
        <a v-if="logged_in === true"  v-on:click="logged_in = false"> <button >Log Out</button></a>
    </div>
</template>
