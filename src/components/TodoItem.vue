<template>
  <div id="TodoItem">
    <div style="padding: 15px;">
      <!-- Main row where the card will iterate -->
      <div class="row">
        <!-- Title and information part -->
        <div class="col-12 info-bar px-0">
          <div class="row align-items-center">
            <div
              class="col-9"
              style="font-size: 15px;font-weight: bold;"
              v-if="on_blur_edit_main === null"
            >
              <span @click="on_blur_edit_main = todo_item.id">
                {{ todo_item.title }}
              </span>
            </div>
            <div class="col-9" v-else>
              <input
                v-model="todo_item.title"
                class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none"
                v-focus
                @keyup.enter="editTodo(todo_item)"
                @focusout="on_blur_edit_main = null"
              />
            </div>
          </div>
        </div>
        <!-- Progress part -->
        <div class="col-12 progress-part px-0">
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
        <div class="col-12 list-bar">
          <div ref="list" v-sortable="{ animation: 200, onUpdate: onSortItem }">
            <div
              class="row py-2 hoverable-icons align-items-center"
              @mouseover="show_icons = index"
              v-for="(item, index) in todo_item.items"
              style="min-height: 46px"
              :key="item.id"
              :id="item.id"
            >
              <div
                class="col-8 px-0"
                :class="on_blur_edit === item.id ? 'd-flex' : ''"
              >
                <label class="checkBoxContainer" style="margin: 14px 20px;">
                  <input
                    v-model="item.complete"
                    type="checkbox"
                    class="h-5 w-5 text-pink-800"
                    tabindex="-1"
                    @change="editTodoItem(item)"
                  />
                  <span class="checkmark"></span>
                </label>
                <span
                  @click="on_blur_edit = item.id"
                  v-if="on_blur_edit !== item.id"
                  :style="
                    item.complete ? 'text-decoration: line-through grey;' : ''
                  "
                >
                  {{ item.title }}
                </span>
                <input
                  v-else-if="on_blur_edit === item.id"
                  v-model="item.title"
                  class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none"
                  v-focus
                  @keyup.enter="
                    editTodoItem(item)
                  "
                  @focusout="on_blur_edit = null"
                />
              </div>
            </div>
          </div>
        </div>
        <!-- Add new list part -->
        <div class="col-12 add-list">
          <div class="row py-2">
            <div class="col-12 px-0" v-if="show_input">
              <input
                type="text"
                class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none"
                v-model="add_todo.title"
                @keyup.enter="
                  addTodo()
                "
                placeholder="Type here"
                v-focus="input_focus"
              />
            </div>
          </div>
          <div class="row py-2">
            <div class="col-6">
              <button
                v-if="show_input"
                class="bg-transparent hover:bg-blue-500 text-blue-700 font-semibold hover:text-white py-2 px-4 border border-blue-500 hover:border-transparent rounded"
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
                class="bg-transparent hover:bg-blue-500 text-blue-700 font-semibold hover:text-white py-2 px-4 border border-blue-500 hover:border-transparent rounded"
                @click="
                  show_input = !show_input;
                  input_focus = true;
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
</template>

<script>
import Sortable from "sortablejs";

export default {
  name: "TodoItem",
  props: {
    todo_item: Object,
    todo_list: Array,
    index: Number
  },
  directives: {
    focus: {
      inserted: function(el) {
        el.focus();
      },
    },
    sortable: {
      inserted: function(el, binding) {
        Sortable.create(el, binding.value || {});
      },
    },
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
      input_focus: true,
      show_icons: null,
      on_blur_edit: null,
      on_blur_edit_main: null,
      delete_item_id: "",
      modal_index: "",
      progressed: "",
      modal_title: "",
      modal_id: ""
    };
  },
  mounted() {
    this.computeWidth();
    return this.todo_item.items.map((item) => {
      item.show = false;
    });
  },
  methods: {
    // compute the width of progress bar
    computeWidth() {
      let list_length =
        this.todo_item.items && this.todo_item.items.length > 0
          ? this.todo_item.items.length
          : 0;
      let checked_length =
        this.todo_item.items && this.todo_item.items.length > 0
          ? this.todo_item.items.filter((item) => item.complete === true).length
          : 0;
      let find_width = (parseInt(checked_length) * 100) / parseInt(list_length);
      return isNaN(Math.round(find_width)) ? 0 : Math.round(find_width);
    },
    // add todo
    addTodo () {
      // this.todo_list[this.index].items.push({ title: this.add_todo.title, complete: false })
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
        self.todo_item.items.map(function(item) {
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
