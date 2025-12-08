<template>
  <div class="login-container">
    <div class="login-box">
      <h2>Connexion</h2>
      
      <form @submit.prevent="handleLogin" v-if="!isRegistering">
        <div class="form-group">
          <label for="email">Email</label>
          <input 
            type="email" 
            id="email" 
            v-model="form.email" 
            required 
            placeholder="votre@email.com"
          />
        </div>
        
        <div class="form-group">
          <label for="password">Mot de passe</label>
          <input 
            type="password" 
            id="password" 
            v-model="form.password" 
            required 
            placeholder="••••••••"
          />
        </div>
        
        <button type="submit" class="login-button" :disabled="loading">
          {{ loading ? 'Connexion en cours...' : 'Se connecter' }}
        </button>
        
        <p class="toggle-auth">
          Pas de compte ? 
          <button type="button" @click="isRegistering = true" class="link-button">
            S'inscrire
          </button>
        </p>
        
        <p v-if="error" class="error">{{ error }}</p>
        <p v-if="success" class="success">{{ success }}</p>
      </form>

      <form @submit.prevent="handleRegister" v-else>
        <div class="form-group">
          <label for="username">Nom d'utilisateur</label>
          <input 
            type="text" 
            id="username" 
            v-model="form.username" 
            required 
            placeholder="votrenomdutilisateur"
          />
        </div>
        
        <div class="form-group">
          <label for="email">Email</label>
          <input 
            type="email" 
            id="email" 
            v-model="form.email" 
            required 
            placeholder="votre@email.com"
          />
        </div>
        
        <div class="form-group">
          <label for="password">Mot de passe</label>
          <input 
            type="password" 
            id="password" 
            v-model="form.password" 
            required 
            placeholder="••••••••"
          />
        </div>
        
        <button type="submit" class="login-button" :disabled="loading">
          {{ loading ? 'Inscription en cours...' : 'S\'inscrire' }}
        </button>
        
        <p class="toggle-auth">
          Vous avez un compte ? 
          <button type="button" @click="isRegistering = false" class="link-button">
            Se connecter
          </button>
        </p>
        
        <p v-if="error" class="error">{{ error }}</p>
        <p v-if="success" class="success">{{ success }}</p>
      </form>
    </div>
  </div>
</template>

<script>
import api from '../api/axios.js';

export default {
  data() {
    return {
      form: {
        username: '',
        email: '',
        password: ''
      },
      error: '',
      success: '',
      loading: false,
      isRegistering: false
    };
  },
  methods: {
    async handleLogin() {
      this.error = '';
      this.success = '';
      this.loading = true;

      try {
        const res = await api.post('/auth', {
          email: this.form.email,
          password: this.form.password
        });
        
        // Stocker le token JWT et le username
        localStorage.setItem('token', res.data.token);
        localStorage.setItem('username', res.data.user?.username || res.data.user?.email || this.form.email);
        localStorage.setItem('user', JSON.stringify(res.data.user));
        
        // Mettre à jour l'en-tête d'autorisation
        api.defaults.headers.common['Authorization'] = `Bearer ${res.data.token}`;
        
        this.success = 'Connexion réussie !';
        
        // Rediriger vers la page d'accueil après 1 seconde
        setTimeout(() => {
          this.$router.push('/');
        }, 1000);
      } catch(err) {
        this.error = err.response?.data?.error || 'Erreur de connexion';
        console.error(err);
      } finally {
        this.loading = false;
      }
    },

    async handleRegister() {
      this.error = '';
      this.success = '';
      this.loading = true;

      try {
        const res = await api.post('/register', {
          username: this.form.username,
          email: this.form.email,
          password: this.form.password
        });
        
        this.success = 'Inscription réussie ! Vous pouvez maintenant vous connecter.';
        
        // Réinitialiser le formulaire et revenir à la connexion
        setTimeout(() => {
          this.isRegistering = false;
          this.form = { username: '', email: '', password: '' };
          this.success = '';
        }, 2000);
      } catch(err) {
        this.error = err.response?.data?.error || 'Erreur lors de l\'inscription';
        console.error(err);
      } finally {
        this.loading = false;
      }
    }
  },
  mounted() {
    // Si l'utilisateur est déjà connecté, rediriger vers la page d'accueil
    if (localStorage.getItem('token')) {
      this.$router.push('/');
    }
  }
};
</script>

<style scoped>
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.login-box {
  background: white;
  padding: 40px;
  border-radius: 10px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  width: 100%;
  max-width: 400px;
}

.login-box h2 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
  font-size: 28px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  color: #555;
  font-weight: 500;
}

.form-group input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.form-group input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.login-button {
  width: 100%;
  padding: 12px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s;
}

.login-button:hover:not(:disabled) {
  transform: translateY(-2px);
}

.login-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.toggle-auth {
  text-align: center;
  margin-top: 20px;
  color: #666;
  font-size: 14px;
}

.link-button {
  background: none;
  border: none;
  color: #667eea;
  cursor: pointer;
  font-weight: 600;
  text-decoration: underline;
  padding: 0;
}

.link-button:hover {
  color: #764ba2;
}

.error {
  color: #e74c3c;
  background-color: #fadbd8;
  padding: 10px;
  border-radius: 5px;
  margin-top: 10px;
  text-align: center;
}

.success {
  color: #27ae60;
  background-color: #d5f4e6;
  padding: 10px;
  border-radius: 5px;
  margin-top: 10px;
  text-align: center;
}

@media (max-width: 480px) {
  .login-box {
    padding: 30px 20px;
  }
}
</style>