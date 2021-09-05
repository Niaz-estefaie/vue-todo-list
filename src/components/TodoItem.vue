<template>
  <div id="TodoItem">
    <div class="p-8">
      <!-- Main row where the card will iterate -->
      <div class="grid lg:grid-cols-3 md:grid-cols-2 sm:grid-cols-1 gap-4">
        <div class="bg-white p-6 rounded-lg">
          <!-- Title and information part -->
          <div class="info-bar px-0">
            <div class="align-items-center">
              <div
                class="font-bold text-center pb-2"
                v-if="on_blur_edit_main === null"
              >
                <span @click="on_blur_edit_main = Item.id">
                  {{ Item.title }}
                </span>
              </div>
              <div v-else>
                <input
                  class="appearance-none mb-2 text-center bg-transparent border border-solid w-full text-gray-700 mr-3 py-1 px-2 leading-tight border-gray-500 border-l-0 border-r-0 border-t-0 focus:outline-none"
                  v-focus
                  v-model="Item.title"
                  @keyup.enter="editTodo(Item)"
                  @focusout="on_blur_edit_main = null"
                />
              </div>
            </div>
          </div>
          <!-- Progress part -->
          <div class="progress-part px-0">
            <div
              class="align-items-center"
              style="display: flex;flex-direction: row;"
            >
              <div
                class="progress"
                :title="computeWidth()"
                style="order: 2;width: 100%"
              >
                <div
                  class="progress-bar"
                  :class="computeWidth() == '100' ? ' bg-success' : ''"
                  :style="'width: ' + computeWidth() + '%'"
                ></div>
              </div>
            </div>
          </div>
          <!-- List of items part -->
          <div class="list-bar">
            <div
              ref="list"
              v-sortable="{ animation: 200, onUpdate: onSortItem }"
            >
              <div
                class="py-2 hoverable-icons items-center"
                @mouseover="show_icons = index"
                v-for="(item, index) in Item.items"
                style="min-height: 46px"
                :key="item.id"
                :id="item.id"
              >
                <div
                  class="px-0 flex"
                >
                  <div class="flex-none">
                    <label class="checkBoxContainer mr-4 after:border-none">
                      <input
                        type="checkbox"
                        v-model="item.complete"
                        class="h-4 w-4 checked:bg-gray-600 checked:border-transparent"
                        tabindex="-1"
                        :style="on_blur_edit === index ? 'vertical-align: middle;' : ''"
                        @change="editTodoItem(item)"
                      />
                    </label>
                  </div>
                  <div class="flex-grow">
                    <span
                      @click="on_blur_edit = index"
                      v-if="on_blur_edit !== index"
                      :style="
                        item.complete ? 'text-decoration: line-through grey;' : ''
                      "
                    >
                      {{ item.title }}
                    </span>
                    <input
                      v-else-if="on_blur_edit === index"
                      v-model="item.title"
                      class="appearance-none bg-transparent w-full text-gray-700 mr-3 py-1 px-2 leading-tight border border-gray-500 border-l-0 border-r-0 border-t-0 focus:outline-none"
                      v-focus
                      @keyup.enter="editTodoItem(item)"
                      @focusout="on_blur_edit = null"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
          <!-- Add new list part -->
          <div class="add-list">
            <div class="py-2">
              <div class="px-0" v-if="show_input">
                <input
                  type="text"
                  class="appearance-none bg-transparent w-full text-gray-700 mr-3 py-1 px-2 border border-solid leading-tight border-gray-500 border-l-0 border-r-0 border-t-0 focus:outline-none"
                  v-model="add_todo.title"
                  @keyup.enter="addTodo()"
                  @focusout="show_input = !show_input"
                  placeholder="Type here"
                  required
                  v-focus="show_input"
                />
              </div>
            </div>
            <div class="py-4">
              <div>
                <button
                  v-if="show_input"
                  class="bg-transparent hover:bg-gray-500 text-gray-700 font-semibold hover:text-white py-2 px-4 border border-gray-500 hover:border-transparent rounded"
                  @click="addTodo()"
                >
                  <i class="la la-plus"></i>
                  Add
                </button>
                <a
                  v-if="show_input"
                  class="m-portlet__nav-link btn m-btn m-btn--hover-default m-btn--icon m-btn--icon-only m-btn--pill"
                  @click="show_input = !show_input"
                >
                  <i class="la la-remove"></i>
                </a>
                <button
                  v-else
                  class="bg-transparent hover:bg-gray-500 text-gray-700 font-semibold hover:text-white py-2 px-4 border border-gray-500 hover:border-transparent rounded"
                  @click="
                    show_input = !show_input
                  "
                >
                  <i class="la la-plus"></i>
                  Add
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Sortable from "sortablejs";

export default {
  name: "TodoItem",
  props: {
    todo_item: Object,
    todo_list: Array,
    index: Number,
  },
  directives: {
    focus: {
      inserted: (el) => {
        el.focus();
      },
    },
    sortable: {
      inserted: (el, binding) => {
        Sortable.create(el, binding.value || {});
      },
    }
  },
  data() {
    return {
      users: [],
      assign_to: [],
      add_todo: {
        title: "",
        deadline: "",
        complete: false,
        user_ids: [],
      },
      assigned: [],
      edit_assigned: [],
      the_item: [],
      edit_todo: {},
      edit_todo_main: {},
      clone_pass_for_delete: {},
      data_object: {
        lists: [],
      },
      sort_items: {
        list: [],
      },
      show_input: false,
      modal_is_open: false,
      is_card: false,
      show_icons: null,
      on_blur_edit: null,
      on_blur_edit_main: null,
      delete_item_id: "",
      modal_index: "",
      progressed: "",
      modal_title: "",
      modal_id: "",
    };
  },
  computed: {
    Lists() {
      return this.todo_list;
    },
    Item() {
      return this.todo_item
    }
  },
  mounted() {
    this.computeWidth();
    return this.Item.items.map((item) => {
      item.show = false;
    });
  },
  methods: {
    // compute the width of progress bar
    computeWidth() {
      let list_length =
        this.Item.items && this.Item.items.length > 0
          ? this.Item.items.length
          : 0;
      let checked_length =
        this.Item.items && this.Item.items.length > 0
          ? this.Item.items.filter((item) => item.complete === true).length
          : 0;
      let find_width = (parseInt(checked_length) * 100) / parseInt(list_length);
      return isNaN(Math.round(find_width)) ? 0 : Math.round(find_width);
    },
    // add todo
    addTodo() {
      this.Lists[this.index].items.push({
        title: this.add_todo.title,
        complete: false,
      });
      this.add_todo.title = "";
    },
    // edit todo item
    editTodoItem(item) {
      console.log(item);
    },
    // select option
    selectedOption(selectedOption) {
      console.log(selectedOption);
    },
    // remove option
    removedOption(removedOption) {
      console.log(removedOption);
    },
    // on sort change
    onSortItem() {
      var self = this;
      var els = self.$refs.list.children;
      self.sort_items.list = [];

      for (var i = 0; i < els.length; i++) {
        var id = els[i].getAttribute("id");
        self.Item.items.map(function(item) {
          if (item.id == id) {
            return self.sort_items.list.push(item.id);
          }
        });
      }

      this.sortItems();
    },
    // Pass item to the modal for edit
    passToModal(item, index) {
      this.modal_index = index;
      this.modal_is_open = true;
      this.modal_id = item.id;
      this.modal_title = item.title;
      this.edit_assigned = [];

      item.assigns.map((item) => {
        return this.edit_assigned.push(item.user ? item.user : item);
      });
    },
    // custom label of options
    customLabel({ persianName, name }) {
      return `${persianName} (${name})`;
    },
    // pass the index for delete
    passId(card, id) {
      this.is_card = card;
      this.delete_item_id = id;
    },
  },
};
</script>

<style>
.checkBoxContainer {
  vertical-align: middle;
}

.checkBoxContainer:after {
  border: none;
}
</style>
