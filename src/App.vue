<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue'

interface IToDoListItem {
    id: number
    content: string
    category: string
    isDone: boolean
    createdAt: Date
}

const todos = ref<IToDoListItem[]>([])
const name = ref('')

const input_content = ref('')

const todos_asc = computed(() => todos.value.sort((a, b) => a.id - b.id))

const addToDo = () => {
    if (input_content.value.trim() === '') {
        console.log('empty')
        return
    }

    const newTodo: IToDoListItem = {
        id: findHighestID() + 1,
        content: input_content.value,
        category: 'general',
        isDone: false,
        createdAt: new Date()
    }

    todos.value.push(newTodo)
}

const removeToDo = (todo: IToDoListItem) => {
    const index = todos.value.findIndex((t) => t.id === todo.id)
    todos.value.splice(index, 1)
}

const findHighestID = () => {
    let highestID = 0
    todos.value.forEach((todo) => {
        if (todo.id > highestID) {
            highestID = todo.id
        }
    })
    return highestID
}

watch(name, (newVal) => {
    localStorage.setItem('name', newVal)
})

watch(
    todos,
    (newVal) => {
        localStorage.setItem('todos', JSON.stringify(newVal))
    },
    { deep: true }
)

onMounted(() => {
    const nameFromStorage = localStorage.getItem('name')
    if (nameFromStorage) {
        name.value = nameFromStorage
    }

    const todosFromStorage = localStorage.getItem('todos')
    if (todosFromStorage) {
        todos.value = JSON.parse(todosFromStorage)
    }
})
</script>

<template>
    <main class="app">
        <section class="greeting">
            <h2 class="title">
                Hello <input type="text" placeholder="Name here" v-model="name" />
            </h2>
        </section>
        <section class="create-todo">
            <h3>Add New</h3>
            <div class="form-container">
                <form @submit.prevent="addToDo">
                    <h4>Whats on your todo list?</h4>
                    <input type="text" placeholder="Item 1" v-model="input_content" />
                    <input type="submit" value="Add todo" />
                </form>
            </div>
        </section>
        <hr />
        <section class="todo-list">
            <h3>Your To-Do list</h3>
            <div class="list">
                <div
                    v-for="todo in todos_asc"
                    :key="todo.id"
                    :class="`todo-item ${todo.isDone && 'done'}`"
                >
                    <label>
                        <input type="checkbox" v-model="todo.isDone" />
                        {{ todo.content }}
                    </label>
                    <div class="actions" :class="{ invisible: !todo.isDone }">
                        <button class="delete" @click="removeToDo(todo)">
                            <span class="material-symbols-outlined"> delete </span>
                        </button>
                    </div>
                </div>
            </div>
        </section>
    </main>
</template>
