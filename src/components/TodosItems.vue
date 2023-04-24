<template>
<div class="todos-list todos-page__list">
  <ul class="todos-list__items">
    <li class="todos-list__item todos-list__item--new">
      <div v-if="error" class="todos-list__error">{{ error }}</div>

      <template v-else>
        <input 
          v-model='newTodoTitle'
          class="todos-list__add-title input"
          type="title"
          placeholder="Title"
        >
  
        <input 
          v-model.number='newTodoUserId'
          class="todos-list__add-id input"
          type="number"
          placeholder="User ID"
        >
        <button
          :disabled="!newTodoTitle || !newTodoUserId"
          class="todos-list__add-button button"
          @click='addToDo'
        >
          Add todo
        </button>
      </template>
    </li>
    <li
      v-for='item in todos'
      :key="item.id"
      class="todos-list__item"
    >
      <span class="todos-list__user">UserId: {{ item.userId }}</span>
      <span class="todos-list__title">Title: {{ item.title }}</span>
      <span class="todos-list__complete">Completed: {{ item.completed ? "&#9989;" : "&#10060;" }}</span>
    </li>
  </ul>
</div>
</template>

<script>
export default {
  name: 'TodosItems',

  props: {
    todos: Array,
    todoLength: Number
  },

  data() {
    return {
      error: '',
      newTodoTitle: '',
      newTodoUserId: ''
    }
  },

  watch: {
    error() {
      if (this.error) {
        setTimeout(() => {
          this.error = ''
        }, 2000)
      }
    }
  },

  methods: {
    async addToDo() {
      try {
        const user = await this.getData(`https://jsonplaceholder.typicode.com/users/${this.newTodoUserId}`)

        if (user && Object.keys(user).length) {
          this.$emit('setTodo', {
            id: this.todoLength + 1,
            userId: this.newTodoUserId,
            title: this.newTodoTitle,
            completed: false
          })
          this.newTodoTitle = ''
          this.newTodoUserId = ''
        }
        else {
          this.error = 'User not found'
        }
      }
      catch (error) {
        console.error(error);
      }
    },

    async getData(url) {
      const response = await fetch(url)
      const json = await response.json()
      return json
    },
  }
}
</script>

<style lang="less">
.todos-list {
  position: relative;

  &__items {
    margin: 0;
    padding: 16px;
    display: flex;
    flex-wrap: wrap;
  }

  &__item {
    display: flex;
    width: calc(50% - 8px);
    flex-direction: column;
    box-sizing: border-box;
    margin: 4px;
    border: 1px solid #519944;
    border-radius: 5px;
    background: #474747;
    padding: 16px;
    justify-content: center;

    @media screen and (min-width: 640px) {
      width: calc(25% - 8px);
    }

    &--new {
      padding: 16px;
      flex-wrap: wrap;
      flex-direction: row;
      align-items: center;
      justify-content: center;
    }
  }

  &__complete {
    white-space: nowrap;
  }

  &__add-id.input {
    margin: 0;
    width: 50%;
    box-sizing: border-box;
  }
  
  &__add-title.input {
    margin: 0;
    margin-bottom: 8px;
  }

  &__add-button {
    width: calc(50% - 8px);
    margin: 0 0 0 8px;
    box-sizing: border-box;
  }

  &__error {
    color: red;
    font-size: 20px;
  }

  @media screen and (max-width: 1024px)  {
     &__add-button,
     &__add-id.input {
      width: 100%;
      margin: 8px 0 0;
     }

     &__add-title.input {
      margin-bottom: 0;
     }
  }
}
</style>