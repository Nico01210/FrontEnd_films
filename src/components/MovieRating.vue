<template>
  <div class="movie-rating">
    <!-- Note IMDb -->
    <div class="rating-item imdb-rating">
      <span class="rating-label">IMDb</span>
      <div class="stars">
        <span v-for="i in 10" :key="i" class="star" :class="{ filled: i <= Math.round(imdbRating) }">★</span>
      </div>
      <span class="rating-value">{{ imdbRating }}/10</span>
    </div>

    <!-- Moyenne des notes utilisateurs -->
    <div v-if="userAverageRating !== null" class="rating-item user-rating">
      <span class="rating-label">Utilisateurs</span>
      <div class="stars">
        <span v-for="i in 10" :key="i" class="star" :class="{ filled: i <= Math.round(userAverageRating) }">★</span>
      </div>
      <span class="rating-value">{{ userAverageRating }}/10</span>
      <span class="rating-count">({{ userRatingCount }})</span>
    </div>

    <!-- Note combinée -->
    <div v-if="userAverageRating !== null" class="rating-item combined-rating">
      <span class="rating-label">Combinée</span>
      <div class="stars">
        <span v-for="i in 10" :key="i" class="star" :class="{ filled: i <= Math.round(combinedRating) }">★</span>
      </div>
      <span class="rating-value">{{ combinedRating }}/10</span>
    </div>
  </div>
</template>

<script>
import api from '../api/axios.js';

export default {
  props: {
    movieId: {
      type: [String, Number],
      required: true
    },
    imdbRating: {
      type: [String, Number],
      default: 0
    }
  },
  data() {
    return {
      userAverageRating: null,
      userRatingCount: 0
    };
  },
  computed: {
    combinedRating() {
      if (this.userAverageRating === null) {
        return parseFloat(this.imdbRating);
      }
      // Moyenne entre IMDb et note utilisateurs
      const imdb = parseFloat(this.imdbRating) || 0;
      return ((imdb + parseFloat(this.userAverageRating)) / 2).toFixed(1);
    }
  },
  methods: {
    async fetchUserRatings() {
      try {
        const res = await api.get(`/movies/${this.movieId}/ratings`);
        const ratings = res.data.member || res.data['hydra:member'] || [];

        if (ratings.length > 0) {
          const sum = ratings.reduce((acc, r) => acc + (r.note || r.score || 0), 0);
          this.userAverageRating = (sum / ratings.length).toFixed(1);
          this.userRatingCount = ratings.length;
        }
      } catch(err) {
        console.error('Erreur lors du chargement des notes utilisateurs:', err);
      }
    }
  },
  mounted() {
    this.fetchUserRatings();
  }
};
</script>

<style scoped>
.movie-rating {
  display: flex;
  gap: 15px;
  align-items: center;
  flex-wrap: wrap;
  margin-top: 10px;
}

.rating-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 8px 12px;
  background: #f8f9fa;
  border-radius: 6px;
  min-width: 100px;
}

.rating-label {
  font-size: 11px;
  font-weight: 600;
  color: #666;
  text-transform: uppercase;
  margin-bottom: 4px;
  letter-spacing: 0.5px;
}

.stars {
  display: flex;
  gap: 2px;
  font-size: 14px;
  margin-bottom: 4px;
}

.star {
  color: #ddd;
}

.star.filled {
  color: #ffc107;
}

.rating-value {
  font-size: 12px;
  font-weight: 700;
  color: #333;
}

.rating-count {
  font-size: 10px;
  color: #999;
  margin-top: 2px;
}

.imdb-rating {
  border-left: 3px solid #f3c500;
}

.user-rating {
  border-left: 3px solid #667eea;
}

.combined-rating {
  border-left: 3px solid #28a745;
}

@media (max-width: 768px) {
  .movie-rating {
    gap: 10px;
  }

  .rating-item {
    min-width: 80px;
    padding: 6px 10px;
  }

  .stars {
    font-size: 12px;
  }

  .rating-value {
    font-size: 11px;
  }
}
</style>
