<template>
<header class="todos-header">
  <div class="todos-header__user-name">{{ userName }}</div>
  <button
    class="todos-header__logout-button button"
    @click="logOut"
  >
    Logout
  </button>
</header>
</template>
<script>
import cookies from 'js-cookie'

export default {
  name: "HeaderOfPage",

  props: { userName: String },

  emits:['updateUser'],

  methods: {
    logOut() {
      cookies.remove('authorized')
      this.$emit('updateUser', null)
    }
  }
}
</script>

<style lang="less">
.todos-header {
  display: flex;
  padding: 8px 24px;
  flex-wrap: wrap;
  background: #474747;
  align-items: center;
  justify-content: space-between;
}
</style>