<template>
  <div class="main-container">
    <authorization-form
      v-if="!userId"
      :users-list="usersList"
      @update-user="updateUser"
    />

    <div v-else class="todos-page">
      <header-of-page
        :userName="userData?.name"
        @update-user="updateUser"
      />

      <multiple-filter 
        :items="todos"
        :search='true'
        :selects='selects'
        @update="updateItems"
      />

      <todos-items 
        v-if="filteredItems?.length"
        :todoLength="todos.length"
        :todos="filteredItems"
        @set-todo="setTodo"
      />
    </div>
  </div>
</template>

<script>

import cookies from 'js-cookie'
import TodosItems from './components/TodosItems.vue'
import HeaderOfPage from './components/HeaderOfPage.vue'
import MultipleFilter from './components/MultipleFilter.vue'
import AuthorizationForm from './components/AuthorizationForm.vue'

export default {
  name: 'App',

  components: {
    TodosItems,
    HeaderOfPage,
    MultipleFilter,
    AuthorizationForm,
  },

  data() {
    return {
      form: {
        phone: '',
        username: ''
      },

      userId: null,
      usersList: [],
      userData: null,

      todos: [],
      storageTodos: [],
      filteredItems: []
    }
  },

  computed: {
    getterUsersIds() {
      return this.usersList.map((e) => {
        return { label: `${e.name} #${e.id}`, value: e.id }
      })
    },

    selects() {
      return [{
        options: [
          { label: 'All',value: null },
          { label: 'Completed', value: true },
          { label: 'Uncompleted', value: false }
        ],
        placeholder: 'Filter by status',
        filter: 'completed'
      },
      {
        options: [{label: 'All users', value: null}].concat(this.getterUsersIds),
        placeholder: 'Filter by user id',
        filter: 'userId'
      }]
    }
  },

  watch: {
    userId() {
      if (this.userId) {
        this.getData(`https://jsonplaceholder.typicode.com/users/${this.userId}`)
          .then(
            (r) => { this.userData = r }
          ).catch(
            (err) => { console.error(err) }
          )
      }
      else {
        this.userData = null
      }
    }
  },

  created() {
    if (cookies.get('authorized')) {
      this.userId = cookies.get('authorized');
    }

    this.getData('https://jsonplaceholder.typicode.com/todos')
    .then(
      (r) => {
        const LSTodos = localStorage.getItem('storageTodos')

        if (LSTodos && Array.isArray(JSON.parse(LSTodos))) {
          this.todos = r.concat(JSON.parse(LSTodos))
        }
        else {
          this.todos = r
        }
      }
    ).catch(
      (err) => { console.error(err) }
    )

    this.getData('https://jsonplaceholder.typicode.com/users')
    .then(
      (r) => { this.usersList = r }
    ).catch(
      (err) => { console.error(err) }
    )
  },

  methods: {
    async getData(url) {
      const response = await fetch(url);
      const json = await response.json();
      return json
    },

    updateUser(payload) {
      this.userId = payload;
    },

    updateItems(payload) {
      this.filteredItems = payload;
    },

    setTodo(payload) {
      const LSTodos = localStorage.getItem('storageTodos')

      let LSTodosParsed = []
      if (LSTodos) {
        LSTodosParsed = JSON.parse(LSTodos)
      }
      LSTodosParsed.push(payload)
      
      if (LSTodosParsed?.length && Array.isArray(LSTodosParsed)) {
        this.todos = this.todos.concat(LSTodosParsed)
        localStorage.setItem('storageTodos', JSON.stringify(LSTodosParsed))
      } else {
        this.todos.push(payload)
        localStorage.setItem('storageTodos', JSON.stringify(LSTodosParsed))
      }
    }
  }
}
</script>

<style lang="less">
.todos-page {
  min-height: 100vh;
  overflow: hidden;
  color: #fff;
  background: #545454;
}
</style>
