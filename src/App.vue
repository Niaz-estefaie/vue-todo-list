<template>
  <div class="container mx-auto">
    <div class="text-center text-salmon">
      <h1 class="py-8 text-6xl">TODO LIST</h1>
    </div>
    <todo-list />
  </div>
</template>

<script>
import TodoList from './components/TodoList.vue'

export default {
  name: 'App',
  components: {
    TodoList
  }
}
</script>

<style>
  body {
    background: #F8D3D0;
    color: #919191;
  }

  .text-salmon {
    color: #F2B0AA;
  }
</style>
