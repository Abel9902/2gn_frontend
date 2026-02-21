<template>
  <div class="admin-login">
    <div class="login-card">
      <h1>Admin Login</h1>
      <p class="subtitle">Sign in to manage the gallery</p>
      <form @submit.prevent="handleLogin">
        <div class="form-row">
          <label>Email</label>
          <input type="email" v-model.trim="email" autocomplete="username" required />
        </div>
        <div class="form-row">
          <label>Password</label>
          <input type="password" v-model="password" autocomplete="current-password" required />
        </div>
        <button class="btn primary" type="submit" :disabled="loading">
          {{ loading ? 'Signing in...' : 'Sign In' }}
        </button>
        <p v-if="error" class="error">{{ error }}</p>
      </form>
      <a class="back-link" href="#home">Back to site</a>
    </div>
  </div>
</template>

<script>
const API_BASE = process.env.VUE_APP_API_BASE || 'http://localhost:3000';

export default {
  name: 'AdminLogin',
  data() {
    return {
      email: '',
      password: '',
      loading: false,
      error: ''
    };
  },
  methods: {
    async handleLogin() {
      this.error = '';
      this.loading = true;
      try {
        const response = await fetch(`${API_BASE}/admin/login`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ email: this.email, password: this.password })
        });

        if (!response.ok) {
          const payload = await response.json().catch(() => null);
          this.error = payload?.message || 'Invalid email or password.';
          return;
        }

        const data = await response.json();
        localStorage.setItem('admin_token', data.access_token);
        this.$emit('login-success');
      } catch (error) {
        console.error('Login failed', error);
        this.error = 'Login failed. Please try again.';
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.admin-login {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f1f5f9;
  padding: 24px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.login-card {
  background: #fff;
  padding: 32px;
  border-radius: 16px;
  width: 100%;
  max-width: 420px;
  box-shadow: 0 20px 40px rgba(15, 23, 42, 0.15);
}

h1 {
  margin: 0 0 8px;
  color: #0f172a;
}

.subtitle {
  margin: 0 0 24px;
  color: #64748b;
}

.form-row {
  display: flex;
  flex-direction: column;
  margin-bottom: 16px;
}

.form-row label {
  font-weight: 600;
  margin-bottom: 6px;
  color: #1f2937;
}

.form-row input {
  border: 1px solid #cbd5f5;
  border-radius: 10px;
  padding: 10px 12px;
  font-size: 14px;
}

.btn {
  width: 100%;
  border: none;
  border-radius: 10px;
  padding: 12px;
  font-weight: 700;
  cursor: pointer;
}

.btn.primary {
  background: #2563eb;
  color: #fff;
}

.btn:disabled {
  opacity: 0.6;
  cursor: default;
}

.error {
  margin-top: 12px;
  color: #ef4444;
  font-weight: 600;
}

.back-link {
  display: inline-block;
  margin-top: 16px;
  color: #2563eb;
  text-decoration: none;
  font-weight: 600;
}
</style>
