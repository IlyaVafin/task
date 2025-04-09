<script lang='ts' setup>
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'

interface Tasks {
	id: number
	title: string
	done: boolean
}

const data = ref<Tasks[]>([])

function loadTasksFromLocalStorage() {
  const savedTasks = localStorage.getItem('tasks')
  if (savedTasks) {
    data.value = JSON.parse(savedTasks)
  }
}


function saveTasksToLocalStorage() {
  localStorage.setItem('tasks', JSON.stringify(data.value))
}

onMounted(async () => {
  loadTasksFromLocalStorage() 
  try {
    if (data.value.length === 0) {
      const response = await axios.get('/public/api/tasks.json')
      data.value = response.data
      saveTasksToLocalStorage()
    }
  } catch (e) {
    console.error("Ошибка при загрузке!", e)
  }
})


watch(data, saveTasksToLocalStorage, { deep: true })
</script>

<template>
  <div v-if="data.length">
    <ul>
      <li v-for="task in data" :key="task.id">
        <input type="checkbox" v-model="task.done" /> 
        <span :class="{ 'completed': task.done }">{{ task.title }}</span>
      </li>
    </ul>
  </div>
  <div v-else>Загрузка...</div>
</template>

<style scoped>
.completed {
  text-decoration: line-through;
  color: grey;
}
</style>
