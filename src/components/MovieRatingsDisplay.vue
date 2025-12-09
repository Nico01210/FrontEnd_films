<template>
  <div class="ratings-display">
    <h3>⭐ Notes des utilisateurs</h3>
    
    <!-- Résumé des ratings -->
    <div v-if="ratingsSummary" class="ratings-summary">
      <div class="summary-item">
        <p class="summary-label">Moyenne</p>
        <div class="summary-stars">
          <span v-for="i in 10" :key="i" class="star" :class="{ filled: i <= Math.round(ratingsSummary.userAverage || ratingsSummary.average) }">★</span>
        </div>
        <p class="summary-value">{{ (ratingsSummary.userAverage || ratingsSummary.average || 0).toFixed(1) }}/10</p>
      </div>
      
      <div class="summary-item">
        <p class="summary-label">Total</p>
        <p class="summary-value large">{{ ratingsSummary.total || ratings.length }}</p>
        <p class="summary-subtext">{{ (ratingsSummary.total || ratings.length) > 1 ? 'votes' : 'vote' }}</p>
      </div>

      <div v-if="combinedRating !== null && combinedRating !== undefined" class="summary-item">
        <p class="summary-label">Combinée</p>
        <div class="summary-stars">
          <span v-for="i in 10" :key="i" class="star" :class="{ filled: i <= Math.round(combinedRating) }">★</span>
        </div>
        <p class="summary-value">{{ parseFloat(combinedRating).toFixed(1) }}/10</p>
      </div>
    </div>

    <!-- Distribution des votes -->
    <div v-if="distribution" class="ratings-distribution">
      <p class="distribution-title">Distribution</p>
      <div v-for="(count, score) in distribution" :key="score" class="distribution-bar">
        <span class="score-label">{{ score }}★</span>
        <div class="bar-container">
          <div class="bar-fill" :style="{ width: getBarWidth(count) }"></div>
        </div>
        <span class="count-label">{{ count }}</span>
      </div>
    </div>

    <!-- Liste des ratings -->
    <div v-if="ratings.length > 0" class="ratings-list">
      <p class="ratings-title">Tous les votes ({{ ratings.length }})</p>
      
      <div v-for="rating in ratings" :key="rating.id" class="rating-item">
        <div class="rating-header">
          <span class="username">👤 {{ rating.user?.username || 'Anonyme' }}</span>
          <span class="date">{{ formatDate(rating.createdAt) }}</span>
        </div>
        
        <div class="rating-content">
          <div class="stars-display">
            <span v-for="i in 10" :key="i" class="star" :class="{ filled: i <= rating.note }">★</span>
          </div>
          <span class="rating-score">{{ rating.note }}/10</span>
        </div>
      </div>
    </div>

    <!-- Message si pas de votes -->
    <div v-else class="no-ratings">
      <p>Aucune note pour ce film pour le moment. Soyez le premier à voter ! 🎬</p>
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
      ratingsSummary: null,
      combinedRating: null,
      ratings: [],
      distribution: null,
      error: null
    };
  },
  methods: {
    async fetchRatings() {
      try {
        // Récupérer TOUTES les ratings du film (sans pagination)
        const res = await api.get(`/movies/${this.movieId}/ratings`, {
          params: { pagination: false }
        });
        const ratingsData = res.data.member || res.data['hydra:member'] || [];
        this.ratings = Array.isArray(ratingsData) ? ratingsData : [];

        console.log('✅ TOUTES les ratings reçues:', this.ratings.length, 'ratings');
        console.log('Contenu complet:', this.ratings);

        // Récupérer le résumé et la note combinée du film
        const movieRes = await api.get(`/movies/${this.movieId}`);
        this.ratingsSummary = movieRes.data.ratingsSummary;
        
        // Extraire la note combinée correctement
        const combinedData = movieRes.data.combinedRating;
        if (typeof combinedData === 'object' && combinedData !== null) {
          this.combinedRating = combinedData.combined || combinedData.average;
        } else {
          this.combinedRating = combinedData;
        }

        console.log('Résumé:', this.ratingsSummary);
        console.log('Note combinée:', this.combinedRating);

        // Calculer la distribution
        this.calculateDistribution();
      } catch(err) {
        this.error = 'Erreur lors du chargement des notes';
        console.error('Erreur:', err);
      }
    },

    calculateDistribution() {
      const dist = {};
      
      // Initialiser la distribution (0-10)
      for (let i = 0; i <= 10; i++) {
        dist[i] = 0;
      }

      // Compter les votes par score
      this.ratings.forEach(rating => {
        if (rating.note >= 0 && rating.note <= 10) {
          dist[rating.note]++;
        }
      });

      // Supprimer les scores sans votes
      Object.keys(dist).forEach(key => {
        if (dist[key] === 0) {
          delete dist[key];
        }
      });

      this.distribution = dist;
    },

    getBarWidth(count) {
      if (!this.ratings.length) return '0%';
      const maxCount = Math.max(...Object.values(this.distribution || {}));
      return ((count / maxCount) * 100) + '%';
    },

    formatDate(dateString) {
      if (!dateString) return 'Date inconnue';
      const date = new Date(dateString);
      return new Intl.DateTimeFormat('fr-FR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      }).format(date);
    }
  },
  mounted() {
    this.fetchRatings();
  }
};
</script>

<style scoped>
.ratings-display {
  margin: 30px 0;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.ratings-display h3 {
  margin: 0 0 25px 0;
  color: #333;
  font-size: 20px;
}

/* Résumé */
.ratings-summary {
  display: flex;
  gap: 30px;
  margin-bottom: 30px;
  flex-wrap: wrap;
  padding: 25px;
  background: #f8f9fa;
  border-radius: 8px;
}

.summary-item {
  flex: 1;
  min-width: 150px;
  text-align: center;
}

.summary-label {
  font-size: 12px;
  font-weight: 600;
  color: #666;
  text-transform: uppercase;
  margin: 0 0 8px 0;
  letter-spacing: 0.5px;
}

.summary-stars {
  display: flex;
  justify-content: center;
  gap: 3px;
  font-size: 20px;
  margin: 8px 0;
}

.star {
  color: #ddd;
}

.star.filled {
  color: #ffc107;
}

.summary-value {
  font-size: 24px;
  font-weight: 700;
  color: #333;
  margin: 8px 0 0 0;
}

.summary-value.large {
  font-size: 32px;
}

.summary-subtext {
  font-size: 12px;
  color: #999;
  margin: 4px 0 0 0;
}

/* Distribution */
.ratings-distribution {
  margin-bottom: 30px;
  padding: 20px;
  background: #f0f0f0;
  border-radius: 8px;
}

.distribution-title {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin: 0 0 15px 0;
  text-transform: uppercase;
}

.distribution-bar {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 12px;
}

.score-label {
  min-width: 35px;
  font-size: 13px;
  font-weight: 600;
  color: #333;
}

.bar-container {
  flex: 1;
  height: 20px;
  background: white;
  border-radius: 4px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #ffc107, #ffa500);
  transition: width 0.3s ease;
}

.count-label {
  min-width: 35px;
  text-align: right;
  font-size: 12px;
  font-weight: 600;
  color: #666;
}

/* Liste des ratings */
.ratings-list {
  margin-top: 30px;
}

.ratings-title {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin: 0 0 15px 0;
  text-transform: uppercase;
}

.rating-item {
  padding: 15px;
  background: #f8f9fa;
  border-radius: 6px;
  margin-bottom: 12px;
  border-left: 3px solid #667eea;
}

.rating-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
  font-size: 13px;
}

.username {
  font-weight: 600;
  color: #333;
}

.date {
  color: #999;
  font-size: 12px;
}

.rating-content {
  display: flex;
  align-items: center;
  gap: 12px;
}

.stars-display {
  display: flex;
  gap: 2px;
  font-size: 16px;
}

.rating-score {
  font-size: 14px;
  font-weight: 700;
  color: #667eea;
  min-width: 45px;
}

/* Message si pas de votes */
.no-ratings {
  text-align: center;
  padding: 30px;
  background: #e8f5e9;
  border-radius: 8px;
  color: #4caf50;
}

.no-ratings p {
  margin: 0;
  font-size: 15px;
}

/* Erreur */
.error {
  color: #dc3545;
  margin-top: 15px;
  font-size: 14px;
  text-align: center;
}

/* Responsive */
@media (max-width: 768px) {
  .ratings-display {
    padding: 20px;
  }

  .ratings-summary {
    flex-direction: column;
    gap: 15px;
    padding: 15px;
  }

  .summary-item {
    min-width: auto;
  }

  .distribution-bar {
    font-size: 12px;
  }

  .rating-item {
    padding: 12px;
  }

  .rating-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 5px;
  }
}
</style>
