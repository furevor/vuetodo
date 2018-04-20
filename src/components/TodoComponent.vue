<template>

  <div id="layout">

    <div class="content">
      <ul class="noteList">
        <li v-for="todo in filteredTodos"
            class="todo"
            :key="todo._id"
            :class="{ completed: todo.completed, editing: todo.editing }">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed" @click="toggleCompletion(todo)">
            <label @dblclick="editTodo(todo)">{{ todo.heading }}</label>
            <button class="buttonsW" @click="removeTodo(todo)">Удалить</button>
            <select name="priority" class="selcls" v-model="todo.priority" @change="onChange(todo)">
              <option value="0">Низкий</option>
              <option value="1">Средний</option>
              <option value="2">Высокий</option>
            </select>
          </div>
          <div class="EditBlock" v-if=todo.editing>
            <input class="edit" type="text"
                   v-model="todo.heading"
                   v-todo-focus="todo == editedTodo"
                   @blur="doneEdit(todo)"
                   @keyup.enter="doneEdit(todo)"
                   @keyup.esc="cancelEdit(todo)">
            <button class="buttonsW" @click="cancelEdit(todo)">Отмена</button>
          </div>
          <div class="dateAlign">
            <input type="date" id="dateInput" class="forDate" v-model="todo.noteDate" @change="onChange(todo)">
          </div>
        </li>
      </ul>

    <div id="addButton" v-if="needToAdd">
      <button class="buttonsW" @click="showAddDiv()">Добавить</button>
    </div>

      <div class="AddNewNote" v-if="needToAdd == false">
        <input class="edit" type = "text" placeholder="Уже придумали себе задачу на сегодня?;)" v-model="newTodo" @keyup.enter="addTodo()">
        <button class="buttonsW" @click="needToAdd = !needToAdd">Отмена</button>
        <button class="buttonsW" @click="addTodo()">Сохранить</button>
      </div>
    </div>
    <div class="importBlock">
      <app-imported-todos @reload="reloadHandler"></app-imported-todos>
    </div>
  </div>

</template>

<script>

// visibility filters
var filters = {
  all: function (todos) {
    return todos
  },
  active: function (todos) {
    return todos.filter(function (todo) {
      return !todo.completed
    })
  },
  completed: function (todos) {
    return todos.filter(function (todo) {
      return todo.completed
    })
  }
}

import axios from 'axios'
import importedTodos from './app-imported-todos.vue';

export default {
  name: 'HelloWorld',
  data() {
    return {
      posts: [],
      todoendpoint: '/api/tasks/',

      todos: [{
        _id: "5a92881aa129700014abf0d6",
        heading: "Здесь должна быть одна задача",
        priority: "0",
        completed: false,
        noteDate: "2018-02-25"
      }/*{"_id":"5a92881aa129700014abf0d6","heading":"Здесь должна быть одна задача","priority":"0","completed":true,"noteDate":"2018-02-25"}*/],
      newTodo: '',
      editedTodo: null,
      visibility: 'all',
      needToAdd: true
    }
  },

  components: {
    'app-imported-todos': importedTodos
  },

  created() {
    this.getAllTasks();
  },

  // computed properties
  // http://vuejs.org/guide/computed.html
  computed: {
    filteredTodos: function () {
      return filters[this.visibility](this.todos)
    },
    remaining: function () {
      return filters.active(this.todos).length
    },
    allDone: {
      get: function () {
        return this.remaining === 0
      },
      set: function (value) {
        this.todos.forEach(function (todo) {
          todo.completed = value
        })
      }
    }
  },

  filters: {
    pluralize: function (n) {
      return n === 1 ? 'item' : 'items'
    }
  },

  methods: {
    getAllTasks() {
      axios.get(this.todoendpoint)
        .then(response => {
          response.data.forEach(function(entry) {
            entry.editing = false;
          });
          this.todos = response.data;
          //console.log('rrr' + response.data);
        })
        .catch(error => {
          console.log('getAllTasks -----error-------');
          console.log(error);
        })
    },
    saveServer(newTodo) {
      axios.post(this.todoendpoint, { priority: 0,
        editing: false,
        completed: false,
        _id: '',
        heading: newTodo,
        noteDate: '2018-04-19' })
        .then(response => {
          response.data.editing = false;
          this.todos.push(response.data);
        })
        .catch(error => {
          console.log('saveServer -----error-------');
          console.log(error);
        })
    },
    deleteServer(todo) {
      axios.delete(this.todoendpoint + '/' + todo._id, { responseType: 'text' })
        .then(response => {
          //this.todos.push(response.data);
          console.log(response.status);
        })
        .catch(error => {
          console.log('deleteServer -----error-------1');
          console.log(error);
        })
    },
    saveChangesServer(todo) {
      axios.put(this.todoendpoint + '/' + todo._id, todo, { responseType: 'text' })
        .then(response => {
          //this.todos.push(response.data);
          console.log(response.status);
        })
        .catch(error => {
          console.log('saveChangesServer -----error-------');
          console.log(error);
        })
    },
    addTodo: function () {
      var value = this.newTodo && this.newTodo.trim()
      if (!value) {
        return
      }
      this.saveServer(value);
      this.newTodo = '';
      this.needToAdd = true;
    },
    showAddDiv: function () {
      this.needToAdd = false;
    },

    removeTodo: function (todo) {
      this.deleteServer(todo);
      this.todos.splice(this.todos.indexOf(todo), 1)
    },

    editTodo: function (todo) {
      this.beforeEditCache = todo.heading;
      this.editedTodo = todo;
      todo.editing = true;
    },

    doneEdit: function (todo) {
      todo.editing = false;
      this.saveChangesServer(todo);
      /*if (!this.editedTodo) {
        return
      }
      this.editedTodo = null;
      todo.title = todo.title.trim()
      if (!todo.title) {
        this.removeTodo(todo)
      }*/

    },

    cancelEdit: function (todo) {
      this.editedTodo = null;
      todo.heading = this.beforeEditCache;
      todo.editing = false;
    },

    toggleCompletion: function (todo) {
      console.log(todo.completed + ' - aaa');
      todo.completed = !todo.completed;
      this.saveChangesServer(todo);
    },
    reloadHandler: function () {
    console.log('hello!');
      this.getAllTasks();
    },
    onChange: function (todo) {
      this.saveChangesServer(todo);
    }

  },
  directives: {
    'todo-focus': function (el, binding) {
      if (binding.value) {
        el.focus()
      }
    }
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #layout {
    display: flex;
    width:1200px;
    margin:0 auto;
    /*border: 1px solid #ededed;
    box-shadow: 0 1px 1px 0 rgba(0,0,0, .2);*/
  }
  #layout div {

    padding: 5px;
    margin:0 auto;
  }
  .importBlock {
    flex: 1 1 30%;
  }
  .content {
    flex: 1 10 70%;
    border: 1px solid #ededed;
    box-shadow: 0 1px 1px 0 rgba(0,0,0, .2);

  }
  div #addButton {
    padding: 5px;
    text-align: center;

  }
  .noteList {
    list-style: none;
    padding: 0;
  }
  .noteList li {

    /*padding: 10px 40px 15px 30px;*/
    /*background: linear-gradient(to left, #5e88d4  0%, white, #5e88d4 );*/
    background: white;
    border-bottom: 1px solid grey;
    color: #506a6b;
    font-size: 20px;
    box-shadow: 0 1px 1px 0 rgba(0,0,0, .2);
    margin-bottom: 5px;
    border: 1px solid #ededed;
  }
  .forDate {
    font-size: 15px;
    padding: 0px 0px 0px 0px;
  }
  #dateInput {
    border: none;
    font-family: arial,sans-serif;
  }
  #dateInput:active {
    border: none;
  }
  .noteList li:last-child {
    border-bottom: none;
  }
  .view {
    position: relative;
    height: 60px;
    text-align: left;
  }
  .dateAlign {
    text-align: left;
  }
  .EditBlock {
    position: relative;
    height: 60px;
  }
  .noteList li.editing .view {
    display: none;
  }
  .noteList li.editing {
    border-bottom: none;
    padding: 0;
  }
  .noteList li.editing .edit {
    /*display: block;*/
    width: 650px;

    /*padding: 13px 17px 12px 17px;*/
    margin: 0 0 0 0px;
    font-size: 20px;
    color: #506a6b;
  }
  .AddNewNote .edit {
    /*display: block;*/
    width: 650px;

    /*padding: 13px 17px 12px 17px;*/
    margin: 0 0 0 0px;
    font-size: 20px;
    color: #506a6b;
  }
  .AddNewNote .buttonsW {
    float: right;
    margin: 5px;
  }
  .noteList li .buttonsW {
    /*
      text-align: center;
      width: 40px;
      /* auto, since non-WebKit browsers doesn't support input styling *
      height: auto;
      position: absolute;
      top: 0;
      bottom: 0;
      margin: auto 0;
      border: none; /* Mobile Safari *
      -webkit-appearance: none;
      appearance: none;*/

    margin-right: 5px;
    float: right;
  }
  .noteList li .buttonsWhileEdit {
    float: right;
    margin-right: 5px;
  }
  .noteList li label {
    margin-left: 45px;
    line-height: 1.2;
    transition: color 0.4s;
  }
  .noteList li.completed label {
    color: #d9d9d9;
    text-decoration: line-through;
  }
  .buttonsW {
    display: inline-block;
    font-family: arial,sans-serif;
    font-size: 16px;
    font-weight: 300;
    color: rgb(68,68,68);
    text-decoration: none;
    user-select: none;
    padding: .2em 1.2em;
    outline: none;
    border: 1px solid rgba(0,0,0,.1);
    border-radius: 2px;
    background: rgb(245,245,245) linear-gradient(#f4f4f4, #f1f1f1);
    transition: all .218s ease 0s;
  }
  .buttonsW:hover {
    color: rgb(24,24,24);
    border: 1px solid rgb(198,198,198);
    background: #f7f7f7 linear-gradient(#f7f7f7, #f1f1f1);
    box-shadow: 0 1px 2px rgba(0,0,0,.1);
  }
  .buttonsW:active {
    color: rgb(51,51,51);
    border: 1px solid rgb(204,204,204);
    background: rgb(238,238,238) linear-gradient(rgb(238,238,238), rgb(224,224,224));
    box-shadow: 0 1px 2px rgba(0,0,0,.1) inset;
  }
  .noteList li .toggle {

    text-align: center;
    width: 40px;
    /* auto, since non-WebKit browsers doesn't support input styling */
    height: auto;
    position: absolute;
    top: 0;
    bottom: 0;
    margin: auto 0;
    border: none; /* Mobile Safari */
    -webkit-appearance: none;
    appearance: none;
  }
  .noteList li .toggle:after {
    content: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="-10 -18 100 135"><circle cx="50" cy="50" r="50" fill="none" stroke="#ededed" stroke-width="3"/></svg>');
  }
  .noteList li .toggle:checked:after {
    content: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="-10 -18 100 135"><circle cx="50" cy="50" r="50" fill="none" stroke="#bddad5" stroke-width="3"/><path fill="#5dc2af" d="M72 25L42 71 27 56l-4 4 20 20 34-52z"/></svg>');
  }
  .selcls {
    margin-right: 3px;
    float: right;
    padding: 4px;
    border: solid 1px #f5f5f5;
    outline: 0;
    box-shadow: rgba(0,0,0, 0.1) 0px 0px 8px;
    -moz-box-shadow: rgba(0,0,0, 0.1) 0px 0px 8px;
    -webkit-box-shadow: rgba(0,0,0, 0.1) 0px 0px 8px;
    border-radius:13px;
    width:100px;
  }
</style>
