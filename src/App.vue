<template>
  <div id="app">
    <input class="input" type="text" v-model="title" @keypress="addTask" placeholder="Task name..."/>
    <Task v-for="task in tasks" :name="task.name" :done="task.done" :key="task._id" :id="task._id" @done="taskDone"/>
  </div>
</template>

<script>
import Task from './components/Task'

export default {
  name: 'app',
  components: {
    Task
  },
  data: () => ({
    tasks: [],
    title: ''
  }),
  mounted() {
    this.getTasks();
  },
  methods: {
    addTask(event) {
      if (this.title && event.keyCode == 13) {
        fetch('/api/tasks/', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({name: this.title})
        })
        .then(() => {
          this.title = '';
          return this.getTasks();
        })
      }
    },
    getTasks() {
      fetch('/api/tasks/')
        .then((response) => response.json())
        .then((json) => { this.tasks = json })
    },
    taskDone(id) {
      console.log(`Task ${id} done`);
      fetch(`/api/tasks/${id}`, {
          method: 'PUT'
        })
        .then(() => {
          return this.getTasks();
        })
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.input {
  height: 36px;
  font-size: 20px;
  width: 200px;
  border-radius: 2px;
  box-shadow: none;
  border: 1px solid #e0e0e0;
  padding: 0 10px;
}
</style>
