<template>
<div class="auth-form-backdrop">
  <div class="auth-form">
    <div class="auth-form__header">Authorization form</div>

    <form class="auth-form__form" @submit.prevent="submitAuthFields">
      <div
        class="auth-form__description"
        :class="{ 'auth-form__description--red': error }"
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
</div>
</template>

<script>
import cookies from 'js-cookie'

export default {
  name: 'AuthorizationForm',

  props: {
    usersList: Array
  },

  emits: {updateUser: Number},

  data() {
    return {
      form: {
        phone: '',
        username: ''
      },
      error: ''
    }
  },

  methods: {
    changeInput(e, type) {
      if (this.error) {
        this.error = '';
      }

      if (type === 'username') {
        this.form.username = this.form.username.replace(/[^A-zА-я/./-]/g, '')
      }

      if (type === 'phone') {
        this.form.phone = this.form.phone.replace(/[^\W0-9\s/x]/gm, '')
      }
    },

    async submitAuthFields() {
      const user = this.usersList.find(el => el.username === this.form.username && el.phone === this.form.phone);

      if (user) {
        this.$emit('updateUser', user.id);
        cookies.set('authorized', user.id, { expires: 1 });
      }
      else {
        this.error = 'Login fail'
      }
    }
  }
}
</script>

<style lang="less">
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
</style>