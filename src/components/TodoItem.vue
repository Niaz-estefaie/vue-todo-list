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
                    tabindex="-1"
                    @change="editTodoItem(item, todo_list.indexOf(todo_item))"
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
                    editTodoItem(item, todo_list.indexOf(todo_item))
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
                  addTodo(todo_list[todo_list.indexOf(todo_item)].id)
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
                class="btn btn-default btn-sm"
                @click="addTodo(todo_list[todo_list.indexOf(todo_item)].id)"
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
                class="btn btn-default btn-sm"
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
    todo_item: [Array, Object],
    todo_list: Array,
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
<style>
.user-avatar {
  border-radius: 50%;
  width: 50px;
  height: 50px;
}

.checkBoxContainer {
  position: relative;
  margin-bottom: 12px;
  cursor: pointer;
  font-size: 10px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

/* Hide the browser's default checkbox */
.checkBoxContainer input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}

/* Create a custom checkbox */
.checkmark {
  margin-left: -13px;
  position: absolute;
  top: 0;
  left: 0;
  height: 20px;
  width: 20px;
  background-color: #eee;
  border-radius: 5px;
  border: 1px solid #80808066;
}

/* On mouse-over, add a grey background color */
.checkBoxContainer:hover input ~ .checkmark {
  background-color: #ccc;
}

/* When the checkbox is checked, add a blue background */
.checkBoxContainer input:checked ~ .checkmark {
  border: 1px solid #34bfa3;
}

/* Create the checkmark/indicator (hidden when not checked) */
.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

/* Show the checkmark when checked */
.checkBoxContainer input:checked ~ .checkmark:after {
  display: block;
}

/* Style the checkmark/indicator */
.checkBoxContainer .checkmark:after {
  left: 6px;
  top: 3px;
  width: 5px;
  height: 10px;
  border: solid #34bfa3;
  border-width: 0 3px 3px 0;
  -webkit-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  transform: rotate(45deg);
}

.hoverable-icons:hover {
  background: #f5f6f7;
}

/* Tooltip css */

.tooltip {
  display: block !important;
  z-index: 10000;
}

.tooltip .tooltip-inner {
  background: white;
  color: #212529;
  border-radius: 5px;
  padding: 5px 10px 4px;
  font-size: 12px;
}

.tooltip .tooltip-arrow {
  width: 0;
  height: 0;
  border-style: solid;
  position: absolute;
  margin: 5px;
  border-color: black;
  z-index: 1;
}

.tooltip[x-placement^="top"] {
  margin-bottom: 5px;
}

.tooltip[x-placement^="top"] .tooltip-arrow {
  border-width: 5px 5px 0 5px;
  border-left-color: transparent !important;
  border-right-color: transparent !important;
  border-bottom-color: transparent !important;
  bottom: -5px;
  left: calc(50% - 5px);
  margin-top: 0;
  margin-bottom: 0;
}

.tooltip[x-placement^="bottom"] {
  margin-top: 5px;
}

.tooltip[x-placement^="bottom"] .tooltip-arrow {
  border-width: 0 5px 5px 5px;
  border-left-color: transparent !important;
  border-right-color: transparent !important;
  border-top-color: transparent !important;
  top: -5px;
  left: calc(50% - 5px);
  margin-top: 0;
  margin-bottom: 0;
}

.tooltip[x-placement^="right"] {
  margin-left: 5px;
}

.tooltip[x-placement^="right"] .tooltip-arrow {
  border-width: 5px 5px 5px 0;
  border-left-color: transparent !important;
  border-top-color: transparent !important;
  border-bottom-color: transparent !important;
  left: -5px;
  top: calc(50% - 5px);
  margin-left: 0;
  margin-right: 0;
}

.tooltip[x-placement^="left"] {
  margin-right: 5px;
}

.tooltip[x-placement^="left"] .tooltip-arrow {
  border-width: 5px 0 5px 5px;
  border-top-color: transparent !important;
  border-right-color: transparent !important;
  border-bottom-color: transparent !important;
  right: -5px;
  top: calc(50% - 5px);
  margin-left: 0;
  margin-right: 0;
}

.tooltip.popover .popover-inner {
  background: #f9f9f9;
  color: black;
  padding: 24px;
  border-radius: 5px;
  box-shadow: -3px 2px 52px -14px rgb(0 0 0 / 37%);
}

.tooltip.popover .popover-arrow {
  border-color: #f9f9f9;
}

.tooltip[aria-hidden="true"] {
  visibility: hidden;
  opacity: 0;
  transition: opacity 0.15s, visibility 0.15s;
}

.tooltip[aria-hidden="false"] {
  visibility: visible;
  opacity: 1;
  transition: opacity 0.15s;
}

.option__image {
  height: 30px;
  width: 30px;
  border-radius: 50%;
}

.avatar {
  border-radius: 50%;
  position: relative;
}

.avatars > li:not(:last-child) {
  margin-right: -0.9rem;
}

.avatars > li:last-child {
  margin-right: -0.9rem;
}

.avatar img {
  display: block;
}

.avatars {
  padding-left: 0;
  list-style: none;
  display: flex;
  margin-bottom: 0;
}

.btn.btn-danger {
  padding: 8px !important;
}

.progress {
  height: 0.4rem !important;
}

a {
  color: #212529;
}
</style>
