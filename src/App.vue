<template>
  <AdminGallery v-if="isAdminRoute && isAuthenticated" @logout="handleLogout" />
  <AdminLogin v-else-if="isAdminRoute" @login-success="handleLoginSuccess" />
  <HomePage v-else />
</template>

<script>
import HomePage from './components/HomePage.vue'
import AdminGallery from './components/AdminGallery.vue'
import AdminLogin from './components/AdminLogin.vue'

export default {
  name: 'App',
  components: {
    HomePage,
    AdminGallery,
    AdminLogin
  },
  data() {
    return {
      isAdminRoute: window.location.hash === '#/admin',
      isAuthenticated: !!localStorage.getItem('admin_token')
    }
  },
  created() {
    window.addEventListener('hashchange', this.handleHashChange);
  },
  beforeUnmount() {
    window.removeEventListener('hashchange', this.handleHashChange);
  },
  methods: {
    handleHashChange() {
      this.isAdminRoute = window.location.hash === '#/admin';
      this.isAuthenticated = !!localStorage.getItem('admin_token');
    },
    handleLoginSuccess() {
      this.isAuthenticated = true;
    },
    handleLogout() {
      localStorage.removeItem('admin_token');
      this.isAuthenticated = false;
      window.location.hash = '#home';
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
