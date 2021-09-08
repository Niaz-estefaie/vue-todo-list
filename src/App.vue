<template>
  <div class="container mx-auto">
    <div class="text-center text-salmon py-8">
      <h1 class="text-6xl inline-block mx-4">TODO LIST</h1>
      <img
        class="w-11 h-auto inline-block align-baseline"
        src="./assets/images/to-do-list.png"
        alt="Alt"
      >
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
