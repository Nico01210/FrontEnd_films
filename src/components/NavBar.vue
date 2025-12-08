<template>
  <nav class="navbar">
    <div class="navbar-content">
      <router-link to="/" class="navbar-brand">
        <h2>🎬 NETFLOX</h2>
      </router-link>
      
      <div class="navbar-actions">
        <div v-if="isLoggedIn" class="user-info">
          <span class="welcome">Bienvenue, {{ username }}</span>
          <button @click="logout" class="btn-logout">Déconnexion</button>
        </div>
        <div v-else class="auth-buttons">
          <router-link to="/login" class="btn-login">Se connecter</router-link>
        </div>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  data() {
    return {
      isLoggedIn: false,
      username: ''
    };
  },
  methods: {
    checkAuth() {
      const token = localStorage.getItem('token');
      const user = localStorage.getItem('username');
      this.isLoggedIn = !!token;
      this.username = user || 'Utilisateur';
    },
    logout() {
      localStorage.removeItem('token');
      localStorage.removeItem('username');
      this.isLoggedIn = false;
      this.username = '';
      this.$router.push('/');
    }
  },
  mounted() {
    this.checkAuth();
  },
  watch: {
    '$route'() {
      this.checkAuth();
    }
  }
};
</script>

<style scoped>
.navbar {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  position: sticky;
  top: 0;
  z-index: 1000;
}

.navbar-content {
  max-width: 1400px;
  margin: 0 auto;
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.navbar-brand {
  text-decoration: none;
  color: white;
}

.navbar-brand h2 {
  margin: 0;
  font-size: 28px;
  font-weight: 700;
}

.navbar-actions {
  display: flex;
  align-items: center;
  gap: 15px;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 15px;
}

.welcome {
  color: white;
  font-size: 16px;
  font-weight: 500;
}

.btn-logout,
.btn-login {
  padding: 10px 20px;
  border-radius: 25px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  text-decoration: none;
  display: inline-block;
}

.btn-logout {
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
  border: 2px solid white;
}

.btn-logout:hover {
  background-color: rgba(255, 255, 255, 0.3);
}

.btn-login {
  background-color: white;
  color: #667eea;
  border: 2px solid white;
}

.btn-login:hover {
  background-color: rgba(255, 255, 255, 0.9);
}

@media (max-width: 768px) {
  .navbar-content {
    flex-direction: column;
    gap: 15px;
  }
  
  .welcome {
    font-size: 14px;
  }
}
</style>
