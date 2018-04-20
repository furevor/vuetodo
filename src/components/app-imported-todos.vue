<template>
  <div id="rightColumn">
    <div class="centerButton">
      <p>
        Импортируйте свои задачи из todoist!
      </p>
      <button class="todoistButton" v-if="projects.length == 0" @click="getTodoistAccountAccess()">Авторизация</button>
    </div>

    <!-- Список с наименованием проектов для выбора!!! -->
    <div id="layout">
      <ul class="projectsList">
        <li v-for="project in projects" @click="onSelect(project)">

          <div class = "view">

            <span class="badge">-</span> <span>{{project.name}}</span>

          </div>

        </li>
      </ul>
      <div class="centerButton">
        <button class="todoistButton" v-if="projects.length > 0" @click="clearServerAccess()">Закрыть</button>
      </div>
    </div>

  </div>
</template>

<script>


  import axios from 'axios'

  export default {
    name: 'app-imported-todos',
    data() {
      return {
        posts: [],
        todoendpoint: '/api/tasks/',
        accessendpoint: '/api/import/access',
        importUrl: '/api/import',
        clearUrl: '/api/import/clear',
        projects: []
      }
    },
    created() {
      // получаем строку параметров из адресной строки, ищем значение параметра code
      let params = new URLSearchParams(window.location.search);
      let secretCode = params.get('code');
      if (secretCode != null) {

        this.getAccessToken(secretCode);
        // очистим параметры из строки браузера
        //window.location.search = "";
      }
    },
    methods: {
      getTodoistAccountAccess() {
        window.location.href = 'https://todoist.com/oauth/authorize?client_id=9772ca018597484abb6ddb5e4a23c966&scope=data:read&state=lunartemple2112&redirect_uri=https://vuetodo-2049.herokuapp.com/';
      },
      onSelect(project) {
        axios.get(this.importUrl + '/' + project.id)
          .then(response => {
            // получим список проектов
            var tempProjects = response.data;
            if (tempProjects.length > 0) {
              tempProjects.forEach(function (item, i, notes) {
                item.noteDate = new Date().toISOString().slice(0, 10);
              });
              this.saveTodosOnServer(tempProjects);
            }
          })
          .catch(error => {
            console.log('onSelect -----error-------');
            console.log(error);
          })
      },
      getProjectsList() {
        axios.get(this.importUrl)
          .then(response => {
            // получим список проектов
            this.projects = response.data;
          })
          .catch(error => {
            console.log('getProjectsList -----error-------');
            console.log(error);
          })
      },
      getAccessToken(secretCode) {
        var body = {
          secretCode: secretCode
        }
        axios.post(this.accessendpoint, body, {responseType: 'text'})
          .then(response => {
            // получим список проектов
            this.getProjectsList();
          })
          .catch(error => {
            console.log('getAccessToken -----error-------');
            console.log(error);
          })
      },
      saveTodosOnServer(todos) {

        axios.post(this.todoendpoint, todos)
          .then(response => {
            this.$emit('reload');
          })
          .catch(error => {
            console.log('saveTodosOnServer -----error-------');
            console.log(error);
          })
      },
      clearServerAccess() {
        this.projects = [];
        axios.get(this.clearUrl)
          .then(response => {

          })
          .catch(error => {
            console.log('clearServerAccess -----error-------');
            console.log(error);
          })
      }
    }

  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #rightColumn {
    width:300px;
    float:right;
    border: 1px solid #ededed;
    margin-right: 100px;
    margin-top: 22px;
    background: white;

  }
  div .centerButton {
    padding: 5px;
    text-align: center;
  }
  .todoistButton {
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
    margin: 0 auto;
  }
  .todoistButton:hover {
    color: rgb(24,24,24);
    border: 1px solid rgb(198,198,198);
    background: #f7f7f7 linear-gradient(#f7f7f7, #f1f1f1);
    box-shadow: 0 1px 2px rgba(0,0,0,.1);
  }
  .todoistButton:active {
    color: rgb(51,51,51);
    border: 1px solid rgb(204,204,204);
    background: rgb(238,238,238) linear-gradient(rgb(238,238,238), rgb(224,224,224));
    box-shadow: 0 1px 2px rgba(0,0,0,.1) inset;
  }
  .selected {
    background-color: #CFD8DC !important;
    color: white;
  }
  .projectsList {
    margin: 0 0 2em 0;
    list-style-type: none;
    padding: 0;
    width: 15em;
  }
  .projectsList li {
    cursor: pointer;
    position: relative;
    left: 0;
    background-color: #EEE;
    margin: .5em;
    padding: .3em 0;
    height: 1.6em;
    border-radius: 4px;
  }
  .projectsList li.selected:hover {
    background-color: #BBD8DC !important;
    color: white;
  }
  .projectsList li:hover {
    color: #607D8B;
    background-color: #DDD;
    left: .1em;
  }
  .projectsList .text {
    position: relative;
    top: -3px;
  }
  .projectsList .badge {
    display: inline-block;
    font-size: small;
    color: white;
    padding: 0.8em 0.7em 0 0.7em;
    background-color: #607D8B;
    line-height: 1em;
    position: relative;
    left: -1px;
    top: -4px;
    height: 1.8em;
    margin-right: .8em;
    border-radius: 4px 0 0 4px;
  }
  .view {
    float: left;
  }
</style>
