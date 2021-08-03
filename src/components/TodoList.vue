<template>
  <div id="TodoList" v-cloak>
    <div ref="list" v-sortable="{ animation: 200, onUpdate: onSort }">
      <div v-for="(item, index) in todo_list" :key="item.id" :id="item.id">
        <TodoItem 
          :todo_item="item"
          v-model:todo_list="todo_list"
          :index="index" 
        />
      </div>
    </div>
    <button
      v-if="show_create_button"
      class="bg-transparent hover:bg-blue-500 text-blue-700 font-semibold hover:text-white py-2 px-4 border border-blue-500 hover:border-transparent rounded"
    >
      New checklist
    </button>
  </div>
</template>

<script>
import TodoItem from "./TodoItem";
import Sortable from "sortablejs";

export default {
  name: "TodoList",
  components: {
    TodoItem,
  },
  directives: {
    sortable: {
      inserted: function(el, binding) {
        Sortable.create(el, binding.value || {});
      },
    },
  },
  props: {
    show_create_button: Boolean,
  },
  data() {
    return {
      card_index: null,
      list_index: 0,
      todo_list: [
        {
          title: "First list",
          items: [
            { id: 1, title: "Title 1", complete: false },
            { id: 2, title: "Title 2", complete: true },
          ],
        },
      ],
      sort_items: {
        list: [],
      },
      add_todo: {
        title: "Todo list",
      },
    };
  },
  methods: {
    // add a new todo list
    addTodoList() {},

    // on sort action
    onSort() {
      var self = this;
      var els = self.$refs.list.children;
      self.sort_items.list = [];

      console.log(els);

      for (var i = 0; i < els.length; i++) {
        var id = els[i].getAttribute("id");
        self.todo_list.map(function(item) {
          if (item.id == id) {
            return self.sort_items.list.push(item.id);
          }
        });
      }

      this.sortList();
    },

    // sort list
    sortList() {},
  },
};
</script>
