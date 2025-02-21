<template>
  <div id="app">
    <Login v-if="!isLoggedIn" @login-success="handleLoginSuccess" />
    <ItemList v-else @logout="handleLogout" />
  </div>
</template>

<script>
import Login from './components/login.vue'
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
  mounted() {
    // 检查 cookie 中是否存在 backendUrl
    const cookies = document.cookie.split(';')
    for (let cookie of cookies) {
      const [name, value] = cookie.trim().split('=')
      if (name === 'backendUrl' && value) {
        this.isLoggedIn = true
        break
      }
    }
  },
  methods: {
    handleLoginSuccess() {
      this.isLoggedIn = true
    },
    handleLogout() {
      // 登出时清除 cookie
      document.cookie = 'backendUrl=;max-age=0;path=/'
      this.isLoggedIn = false
    }
  }
}
</script>