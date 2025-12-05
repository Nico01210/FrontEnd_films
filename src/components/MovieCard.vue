<template>
  <div class="movie-card">
    <div v-if="posterUrl" class="movie-poster-container">
      <img :src="posterUrl" :alt="movie.title" class="movie-poster" />
    </div>
    <div v-else class="movie-poster-placeholder">
      <span>🎬</span>
    </div>
    <div class="movie-info">
      <h3>{{ movie.title }}</h3>
      <p class="movie-year" v-if="releaseYear">{{ releaseYear }}</p>
      <p class="movie-rating" v-if="rating">⭐ {{ rating }}/10</p>
      <p class="movie-description" v-if="description">{{ description }}</p>
      <p class="movie-genres" v-if="genres">🎭 {{ genres }}</p>
      
      <!-- Affichage de toutes les propriétés disponibles pour debug -->
      <div class="movie-extra" v-if="hasExtraInfo">
        <p v-if="movie.director">🎬 {{ movie.director }}</p>
        <p v-if="movie.country">🌍 {{ movie.country }}</p>
        <p v-if="movie.actors">👥 {{ movie.actors }}</p>
      </div>
      
      <router-link :to="`/movie/${movieId}`" class="view-details">Voir les détails</router-link>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    movie: {
      type: Object,
      required: true
    }
  },
  computed: {
    movieId() {
      // Gérer différents formats d'ID (@id pour API Platform)
      return this.movie.id || this.movie['@id']?.split('/').pop();
    },
    posterUrl() {
      return this.movie.posterUrl || this.movie.poster || this.movie.posterPath || this.movie.poster_path;
    },
    releaseYear() {
      const date = this.movie.releaseDate || this.movie.release_date || this.movie.releasedAt;
      return date ? new Date(date).getFullYear() : null;
    },
    description() {
      const desc = this.movie.description || this.movie.overview || this.movie.synopsis;
      if (!desc) return null;
      // Tronquer la description à 150 caractères pour la carte
      return desc.length > 150 ? desc.substring(0, 150) + '...' : desc;
    },
    rating() {
      const rate = this.movie.rating || this.movie.vote_average || this.movie.score;
      if (!rate) return null;
      return typeof rate === 'number' ? rate.toFixed(1) : rate;
    },
    genres() {
      if (this.movie.genres) {
        if (Array.isArray(this.movie.genres)) {
          return this.movie.genres.map(g => {
            if (typeof g === 'string') return g;
            if (typeof g === 'object' && g !== null) {
              return g.name || g.label || JSON.stringify(g);
            }
            return String(g);
          }).join(', ');
        }
        if (typeof this.movie.genres === 'string') {
          return this.movie.genres;
        }
      }
      return null;
    },
    hasExtraInfo() {
      return this.movie.director || this.movie.country || this.movie.actors;
    }
  },
  mounted() {
    // Debug temporaire pour voir la structure des données
    console.log('Movie data:', this.movie);
    console.log('Propriétés disponibles:', Object.keys(this.movie));
  }
};
</script>

<style scoped>
.movie-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.3s, box-shadow 0.3s;
  background: white;
}

.movie-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.movie-poster-container {
  width: 100%;
  height: 300px;
  overflow: hidden;
  background-color: #f0f0f0;
}

.movie-poster {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.movie-poster-placeholder {
  width: 100%;
  height: 300px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 80px;
}

.movie-info {
  padding: 15px;
}

.movie-info h3 {
  margin: 0 0 10px 0;
  font-size: 1.2em;
  color: #333;
}

.movie-year {
  color: #666;
  font-size: 0.9em;
  margin: 5px 0;
}

.movie-rating {
  color: #f39c12;
  font-size: 0.9em;
  font-weight: 600;
  margin: 5px 0;
}

.movie-description {
  color: #555;
  font-size: 0.9em;
  margin-bottom: 15px;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.movie-genres {
  color: #324dc4;
  font-size: 0.85em;
  font-weight: 600;
  margin: 10px 0;
}

.movie-extra {
  margin: 10px 0;
}

.movie-extra p {
  font-size: 0.85em;
  color: #666;
  margin: 5px 0;
}

.view-details {
  display: inline-block;
  padding: 8px 16px;
  background-color: #324dc4;
  color: white;
  text-decoration: none;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.view-details:hover {
  background-color: #101e5c;
}
</style>