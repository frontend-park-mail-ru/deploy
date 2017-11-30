<template>
  <h2 :class="['task', done ? 'done' : '']" @click="click">
    {{name}}
  </h2>
</template>

<script>
export default {
  name: 'Task',
  props: [
    'name',
    'done',
    'id'
  ],
  methods: {
    click() {
      this.$emit('done', this.id)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

.task {
  cursor: pointer;
}

.done {
  color: #42b983;
  text-decoration: line-through;
}
</style>
