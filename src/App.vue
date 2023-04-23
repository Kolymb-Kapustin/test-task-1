<template>
  <div class="main-container">
    <AuthorizationForm v-if="!authorized" />
    <!-- <div v-if='!authorized' class="auth-form-backdrop">
      <div class="auth-form">
        <div class="auth-form__header">Authorization form</div>

        <form class="auth-form__form" @submit.prevent="submitAuthFields">
          <div
            class="auth-form__description"
            :class="{'auth-form__description--red': error}"
          >
            {{ error ? error : 'Enter your details' }}
          </div>

          <input
            v-model="form.username"
            @input="changeInput($event, 'username')"
            type="text"
            id="username"
            name="username"
            placeholder="Username"
            class="auth-form__input"
          >

          <input 
            v-model="form.phone"
            @input="changeInput($event, 'phone')"
            type="tel"
            id="phone"
            name="phone"
            placeholder="Phone Number"
            class="auth-form__input"
          >

          <button
              :disabled="!this.form.username || !this.form.phone"
              type="submit"
              class="auth-form__button button"
            >
              Login
            </button>
        </form>
      </div>
    </div> -->

    <div v-else class="todos-page">
      <header class="todos-page__header">
        <div class="todos-page__user-name">{{ userData?.name }}</div>
        <button
          class="todos-page__logout-button button"
          @click="logOut"
        >
          Logout
        </button>
      </header>

      <!-- <div class="filter">
        <div class="filter__select-status">
          <VueMultiselect
            :model-value="selectedStatus"
            :options="Statuses"
            :searchable="false"
            :preselectFirst="true"
            @update:modelValue="selectedStatus = $event"
            placeholder="Filter by status"
          />
        </div>
        <div class="filter__select-user-id">
          <VueMultiselect
            :model-value="selectedUserIdFilter"
            :options="userIdFilterOptions"
            :searchable="false"
            :preselectFirst="true"
            @update:modelValue="selectedUserIdFilter = $event"
            placeholder="Filter by user id"
          />
        </div>
        <div class="filter__search-title">
          <input
            class="filter__search-input"
            v-model="searchTitle"
            type="text"
            placeholder="Search"
          >
        </div>
      </div>

      <div
        v-if="todos?.length"
        class="todos-list todos-page__list"
      >
        <ul class="todos-list__items">
          <li
            v-for='item in filteredTodos'
            :key="item.id"
            class="todos-list__item"
          >
            <span class="todos-list__user">UserId: {{ item.userId }}</span>
            <span class="todos-list__title">Title: {{ item.title }}</span>
            <span class="todos-list__complete">Completed: {{ item.completed ? "&#9989;" : "&#10060;" }}</span>
          </li>
        </ul>
      </div> -->
    </div>
  </div>
</template>

<script>
import cookies from 'js-cookie'
// import VueMultiselect from 'vue-multiselect'
import AuthorizationForm from './components/AuthorizationForm.vue'

export default {
  name: 'App',

  components: {
    // VueMultiselect,
    AuthorizationForm
  },

  data() {
    return {
      form: {
        phone: '',
        username: ''
      },

      error: '',

      usersList: [],
      userData: null,
      userId: cookies.get('authorized'),

      todos: [],
      selectedStatus: '',
      Statuses: ['All', 'Completed', 'Uncompleted'],

      selectedUserIdFilter: '',
      userIdFilterOptions: ['All users'],

      searchTitle: ''
    }
  },

  computed: {
    authorized() {
      return Boolean(this.userId);
    },

    filteredTodos() {
      let result = this.todos;

      if (this.selectedStatus?.match(/Completed|Uncompleted/gm)) {  
        let completed = this.selectedStatus === 'Completed' ? true : false
        result = result.filter(el => el.completed === completed)
      }

      if (this.selectedUserIdFilter && this.selectedUserIdFilter !== 'All users') {
        result = result.filter(el => el.userId === this.selectedUserIdFilter)
      }

      if (this.searchTitle.length) {
        result = result.filter(el => el.title.includes(this.searchTitle))
      }

      return result;
    },

    getterUsersIds() {
      return this.usersList.map((e) => e.id)
    }
  },

  watch: {
    getterUsersIds() {
      this.userIdFilterOptions =  this.userIdFilterOptions.concat(this.getterUsersIds)
    }
  },

  created() {
    if (this.authorized) {
      this.getData(`https://jsonplaceholder.typicode.com/users/${this.userId}`)
      .then(
        (r) => { this.userData = r }
      ).catch(
        (err) => { console.error(err)}
      )

      this.getData('https://jsonplaceholder.typicode.com/todos')
      .then(
        (r) => { this.todos = r }
      ).catch(
        (err) => { console.error(err) }
      )

      this.getData('https://jsonplaceholder.typicode.com/users')
      .then(
        (r) => { this.usersList = r }
      ).catch(
        (err) => { console.error(err) }
      )
    }
  },

  methods: {
    async submitAuthFields() {
      this.usersList = await this.getData('https://jsonplaceholder.typicode.com/users');
      const user = this.usersList.find(el => el.username === this.form.username && el.phone === this.form.phone);

      if (user) {
        this.userData = user;
        this.userId = user.id;
        cookies.set('authorized', user.id, {expires: 1});
      }
      else {
        this.error = 'Login fail'
      }
    },

    logOut() {
      cookies.remove('authorized')
      this.userId = null;
      this.userData = null;
    },

    async getData(url) {
      const response = await fetch(url);
      const json = await response.json();
      return json
    }
  }
}
</script>

<style lang="less">
.filter {
  display: flex;
  flex-wrap: wrap;
  padding: 0 16px;
  margin-top: 16px;

  &__select-status {
    display: inline-block;
    min-width: 300px;
  }

  &__select-user-id {
    min-width: 250px;
    margin-left: 16px;
    display: inline-block;
  }

  &__search-input {
    width: 100%;
    height: 40px;
    padding: 0 12px;
    margin: 0 16px;
    outline: none;
    color: #fff;
    border-radius: 5px;
    box-sizing: border-box;
    background: #474747;
    border: 1px solid #519944;
    font-size: 16px;

    &::placeholder {
      color: #C4C4C4;
    }
  }
  
  .multiselect {
    &__content-wrapper {
      color: #fff;
      border-radius: 5px;
      background: #474747;
      border: 1px solid #519944;
    }

    &__option--selected {
      color: #fff;
      background: #545454;
    }

    &__tags {
      cursor: pointer;
      color: #fff;
      padding-top: 10px;
      border-radius: 5px;
      background: #474747;
      border: 1px solid #519944;
    }
  }
}

.auth-form {
  top: 25%;
  width: 447px;
  margin: 0 auto;
  max-width: 95%;
  border-radius: 5px;
  position: relative;
  background: #C4C4C4;

  &__header {
    padding: 15px;  
    font-size: 17px;
    line-height: 21px;
    text-align: center;
    background: #A5A5A5;
    border-radius: 5px 5px 0 0;
  }

  &__form {
    display: flex;
    flex-direction: column;
  }

  &__description {
    font-size: 15px;
    margin: 15px auto 14px 25px;

    &--red {
      color: red;
    }
  }

  &__input {
      border: none;
      padding: 10px;
      font-size: 17px;
      margin: 0 25px 20px;
      border-radius: 5px;
      background: #fff;
  }

  &__button {
    margin: 5px 25px 30px;
  }

  &-backdrop {
    width: 100vw;
    height: 100vh;
    position: absolute;
    background: #545454;
  }
}

.todos-page {
  min-height: 100vh;
  overflow: hidden;
  color: #fff;
  background: #545454;

  &__header {
    display: flex;
    padding: 8px 16px;
    flex-wrap: wrap;
    background: #474747;
    align-items: center;
    justify-content: space-between;
  }
}

.todos-list {
  position: relative;

  &__items {
    margin: 16px 0 0;
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
    padding: 8px 16px;

    @media screen and (min-width: 640px) {
      width: calc(25% - 8px);
    }
  }

  &__complete {
    white-space: nowrap;
  }
}
</style>
