<template>
  <div class="star-rating">
    <h3>⭐ Noter ce film</h3>
    
    <div v-if="isLoggedIn" class="rating-container">
      <!-- Affichage de la note actuelle de l'utilisateur -->
      <div v-if="userRating && !editMode" class="user-rating-display">
        <p>Votre note : <strong>{{ userRating }}/10</strong></p>
        <div class="stars-display">
          <span v-for="i in 10" :key="i" class="star" :class="{ filled: i <= userRating }">★</span>
        </div>
        <div class="rating-actions">
          <button @click="startEdit" class="btn-edit-rating">✏️ Modifier ma note</button>
          <button @click="deleteRating" class="btn-delete-rating">🗑️ Supprimer ma note</button>
        </div>
      </div>
      
      <!-- Formulaire pour noter ou modifier -->
      <div v-if="!userRating || editMode" class="rating-form">
        <p v-if="!userRating">Donnez une note à ce film</p>
        <p v-else>Modifiez votre note</p>
        <div class="stars-input">
          <span 
            v-for="i in 10" 
            :key="i" 
            class="star clickable" 
            :class="{ hovered: i <= hoverRating, selected: i <= selectedRating }"
            @click="selectedRating = i"
            @mouseover="hoverRating = i"
            @mouseleave="hoverRating = 0"
          >
            ★
          </span>
        </div>
        <div v-if="selectedRating > 0" class="rating-display">
          <p>{{ selectedRating }}/10</p>
          <div class="form-actions">
            <button @click="submitRating" :disabled="submitting" class="btn-submit">
              {{ submitting ? 'Envoi...' : 'Valider' }}
            </button>
            <button v-if="editMode" @click="cancelEdit" class="btn-cancel">Annuler</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Message pour utilisateur non connecté -->
    <div v-else class="login-prompt">
      <p>Connectez-vous pour noter ce film</p>
      <router-link to="/login" class="btn-login">Se connecter</router-link>
    </div>
    
    <p v-if="error" class="error">{{ error }}</p>
  </div>
</template>

<script>
import api from '../api/axios.js';

export default {
  props: {
    movieId: {
      type: [String, Number],
      required: true
    }
  },
  data() {
    return {
      isLoggedIn: false,
      currentUsername: null,
      userRating: null,
      userRatingObject: null,
      selectedRating: 0,
      hoverRating: 0,
      submitting: false,
      error: null,
      editMode: false
    };
  },
  methods: {
    checkAuth() {
      const token = localStorage.getItem('token');
      this.isLoggedIn = !!token;
      this.currentUsername = localStorage.getItem('username');
    },
    
    async fetchUserRating() {
      try {
        // Récupérer la note de l'utilisateur
        const res = await api.get(`/movies/${this.movieId}/ratings`);
        const ratings = res.data.member || res.data['hydra:member'] || [];
        
        // Réinitialiser d'abord
        this.userRating = null;
        this.userRatingObject = null;
        
        // Trouver la note de l'utilisateur actuel
        if (this.currentUsername && ratings.length > 0) {
          const myRating = ratings.find(r => r.user?.username === this.currentUsername);
          if (myRating) {
            this.userRating = myRating.note || myRating.score;
            this.userRatingObject = myRating; // Stocker l'objet entier avec l'ID
            console.log('Note de l\'utilisateur trouvée:', this.userRating);
            console.log('Objet rating:', this.userRatingObject);
          }
        }
      } catch(err) {
        console.error('Erreur lors du chargement de la note utilisateur:', err);
      }
    },
    
    async submitRating() {
      if (this.selectedRating === 0) return;
      
      this.submitting = true;
      this.error = null;
      
      try {
        if (this.editMode && this.userRatingObject) {
          // Mode modification : utiliser l'ID stocké
          const ratingId = this.userRatingObject.id;
          console.log('Modification de la note:', ratingId);
          
          await api.patch(`/ratings/${ratingId}`, { note: this.selectedRating }, {
            headers: { 'Content-Type': 'application/merge-patch+json' }
          });
          this.editMode = false;
        } else {
          // Mode création : POST nouvelle note
          const payload = {
            note: this.selectedRating,
            movie: `/api/movies/${this.movieId}`
          };
          
          console.log('Envoi de la note:', payload);
          await api.post(`/movies/${this.movieId}/ratings`, payload);
        }
        
        this.userRating = this.selectedRating;
        this.selectedRating = 0;
        
        // Recharger les notes
        await this.fetchUserRating();
        
        // Émettre un événement pour rafraîchir les ratings affichés
        this.$emit('rating-updated');
      } catch(err) {
        this.error = err.response?.data?.detail || 'Erreur lors de l\'ajout de la note';
        console.error(err);
        console.error('Détails erreur:', err.response?.data);
      } finally {
        this.submitting = false;
      }
    },
    
    editRating() {
      this.editMode = true;
      this.selectedRating = this.userRating;
    },

    startEdit() {
      this.editMode = true;
      this.selectedRating = this.userRating;
    },

    cancelEdit() {
      this.editMode = false;
      this.selectedRating = 0;
    },
    
    async deleteRating() {
      if (!confirm('Êtes-vous sûr de vouloir supprimer votre note ?')) {
        return;
      }
      
      this.submitting = true;
      this.error = null;
      
      try {
        if (!this.userRatingObject) {
          throw new Error('Note non trouvée');
        }
        
        const ratingId = this.userRatingObject.id;
        console.log('Suppression de la note:', ratingId);
        
        await api.delete(`/ratings/${ratingId}`);
        
        this.userRating = null;
        this.userRatingObject = null;
        this.selectedRating = 0;
        this.editMode = false;
        
        // Émettre un événement pour rafraîchir les ratings affichés
        this.$emit('rating-updated');
      } catch(err) {
        this.error = err.response?.data?.detail || err.message || 'Erreur lors de la suppression';
        console.error('Erreur:', err);
      } finally {
        this.submitting = false;
      }
    }
  },
  mounted() {
    this.checkAuth();
    this.fetchUserRating();
  }
};
</script>

<style scoped>
.star-rating {
  margin: 30px 0;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.star-rating h3 {
  margin: 0 0 20px 0;
  color: #333;
  font-size: 20px;
}

.rating-container {
  margin-bottom: 20px;
}

.user-rating-display {
  padding: 20px;
  background: #f0f8ff;
  border-left: 4px solid #667eea;
  border-radius: 4px;
  margin-bottom: 20px;
}

.user-rating-display p {
  margin: 0 0 10px 0;
  color: #333;
}

.rating-form {
  padding: 20px;
  background: #f8f9fa;
  border-radius: 4px;
  margin-bottom: 20px;
}

.rating-form p {
  margin: 0 0 15px 0;
  color: #555;
}

.stars-input,
.stars-display {
  display: flex;
  gap: 5px;
  font-size: 40px;
  margin: 10px 0;
}

.star {
  cursor: pointer;
  color: #ddd;
  transition: color 0.2s;
}

.star.filled {
  color: #ffc107;
}

.star.clickable:hover,
.star.hovered {
  color: #ffc107;
}

.star.selected {
  color: #ffc107;
}

.rating-display {
  margin-top: 15px;
}

.rating-display p {
  margin: 0 0 10px 0;
  font-size: 18px;
  font-weight: bold;
  color: #667eea;
}

.btn-edit-rating,
.btn-delete-rating,
.btn-submit,
.btn-cancel,
.btn-login {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn-submit {
  background-color: #667eea;
  color: white;
}

.btn-submit:hover:not(:disabled) {
  background-color: #5568d3;
}

.btn-submit:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.btn-edit-rating {
  background-color: #ffc107;
  color: #333;
}

.btn-edit-rating:hover {
  background-color: #e0a800;
}

.btn-delete-rating {
  background-color: #dc3545;
  color: white;
}

.btn-delete-rating:hover {
  background-color: #c82333;
}

.btn-cancel {
  background-color: #6c757d;
  color: white;
}

.btn-cancel:hover {
  background-color: #5a6268;
}

.rating-actions,
.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 15px;
  flex-wrap: wrap;
}

.average-rating {
  margin-top: 30px;
  padding: 20px;
  background: #f0f0f0;
  border-radius: 4px;
}

.average-rating p {
  margin: 0 0 10px 0;
  color: #333;
}

.login-prompt {
  text-align: center;
  padding: 30px;
  background: #f8f9fa;
  border-radius: 4px;
  margin-bottom: 20px;
}

.login-prompt p {
  margin: 0 0 15px 0;
  color: #666;
}

.btn-login {
  display: inline-block;
  padding: 10px 24px;
  background-color: #667eea;
  color: white;
  text-decoration: none;
}

.btn-login:hover {
  background-color: #5568d3;
}

.error {
  color: #dc3545;
  margin-top: 10px;
  font-size: 14px;
}

@media (max-width: 768px) {
  .stars-input,
  .stars-display {
    font-size: 30px;
  }
}
</style>