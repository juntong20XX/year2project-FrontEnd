<template>
  <div id="app">
    <Login v-if="!isLoggedIn" @login-success="handleLoginSuccess" />
    <ItemList v-else @logout="handleLogout" />
  </div>
</template>

<script>
import Login from './components/Login.vue'
import ItemList from './components/control-page.vue'

export default {
  name: 'App',
  components: {
    Login,
    ItemList
  },
  data() {
    return {
      isLoggedIn: false
    }
  },
  methods: {
    handleLoginSuccess() {
      this.isLoggedIn = true
    },
    handleLogout() {
      this.isLoggedIn = false
    }
  }
}
</script>