<template>
  <div class="movie-detail" v-if="movie">
    
    <div class="movie-content">
      <div v-if="posterUrl" class="movie-poster-container">
        <img :src="posterUrl" :alt="movieTitle" class="movie-poster" />
      </div>
      <div v-else class="movie-poster-placeholder">
        <span>🎬</span>
      </div>
      
      <div class="movie-info">
        <h1>{{ movieTitle }}</h1>
        
        <div class="movie-meta">
          <p v-if="releaseDate" class="release-date">
            <strong>📅 Date de sortie :</strong> {{ formatDate(releaseDate) }}
          </p>
          <p v-if="duration" class="duration">
            <strong>⏱️ Durée :</strong> {{ duration }} min
          </p>
          <p v-if="rating" class="rating">
            <strong>⭐ Note :</strong> {{ rating }}/10
          </p>
        </div>
        
        <p class="description" v-if="description">{{ description }}</p>
        
        <div class="additional-info">
          <p v-if="director"><strong>🎬 Réalisateur :</strong> {{ director }}</p>
          <p v-if="genres"><strong>🎭 Genres :</strong> {{ genres }}</p>
          <p v-if="actors"><strong>👥 Acteurs :</strong> {{ actors }}</p>
          <p v-if="country"><strong>🌍 Pays :</strong> {{ country }}</p>
          <p v-if="language"><strong>🗣️ Langue :</strong> {{ language }}</p>
        </div>
        
        <!-- Debug: Afficher toutes les propriétés disponibles -->
        <details class="debug-info">
          <summary>🔍 Données brutes (debug)</summary>
          <pre>{{ movie }}</pre>
        </details>
      </div>
    </div>
  </div>
  <div v-else class="loading">
    <p>Chargement...</p>
  </div>
      <button @click="$router.back()" class="back-button">← Retour</button>

</template>

<script>
import api from '../api/axios.js';

export default {
  data() {
    return {
      movie: null
    };
  },
  computed: {
    movieTitle() {
      return this.movie?.title || 'Titre inconnu';
    },
    posterUrl() {
      return this.movie?.poster;
    },
    releaseDate() {
      return this.movie?.year; // ton API n'a que 'year'
    },
    description() {
      return this.movie?.plot;
    },
    director() {
      if (this.movie?.directors?.length) {
        // Retourne tous les réalisateurs, séparés par une virgule si plusieurs
        return this.movie.directors.map(d => d.fullName).join(', ');
      }
      return null;
    },
    rating() {
      return this.movie?.imdb?.rating || this.movie?.tomatoes?.rating || null;
    },
    genres() {
      const genresData = this.movie?.genres;
      if (!genresData) return null;
      return genresData.map(g => g.label).join(', ');
    },
    actors() {
      const cast = this.movie?.castMembers;
      if (!cast) return null;
      return cast.slice(0, 5).map(a => a.fullName).join(', ');
    },
    country() {
      // Gérer différents formats de pays
      if (this.movie?.countries?.length) {
        return this.movie.countries.map(c => c.label || c.name || c).join(', ');
      }
      if (this.movie?.country) {
        if (typeof this.movie.country === 'string') return this.movie.country;
        if (this.movie.country.label || this.movie.country.name) {
          return this.movie.country.label || this.movie.country.name;
        }
      }
      return this.movie?.origin_country || null;
    },
  },
  methods: {
    async fetchMovie() {
      try {
        const res = await api.get(`/movies/${this.$route.params.id}`);
        console.log('Données du film:', res.data);
        this.movie = res.data;
      } catch(err) {
        console.error('Erreur lors du chargement du film:', err);
        alert('Impossible de charger les détails du film');
        this.$router.push('/');
      }
    },
    formatDate(date) {
      if (!date) return '';
      return new Date(date, 0).toLocaleDateString('fr-FR', {
        year: 'numeric',
      });
    }
  },
  mounted() {
    this.fetchMovie();
  }
};
</script>

<style scoped>
.movie-detail {
  max-width: 1200px;
  margin: 20px auto;
  padding: 20px;
}

.back-button {
  padding: 10px 20px;
  background-color: #324dc4;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  margin-left: auto;
  display: block;
  margin-bottom: 20px;
  margin-right: 450px;
}

.back-button:hover {
  background-color: #101e5c;
}

.movie-content {
  display: flex;
  gap: 30px;
  background: rgb(218, 213, 213);
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.movie-poster-container {
  width: 300px;
  min-width: 300px;
  height: 450px;
  border-radius: 8px;
  overflow: hidden;
  background-color: #f0f0f0;
}

.movie-poster {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.movie-poster-placeholder {
  width: 300px;
  min-width: 300px;
  height: 450px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 100px;
}

.movie-info {
  flex: 1;
}

.movie-info h1 {
  margin: 0 0 20px 0;
  color: #333;
  font-size: 2em;
}

.movie-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 8px;
}

.movie-meta p {
  margin: 0;
  color: #555;
  font-size: 0.95em;
}

.description {
  line-height: 1.8;
  color: #555;
  margin-bottom: 30px;
  font-size: 1.05em;
  text-align: justify;
}

.additional-info {
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.additional-info p {
  margin: 12px 0;
  color: #555;
  line-height: 1.6;
}

.debug-info {
  margin-top: 30px;
  padding: 15px;
  background-color: #f0f0f0;
  border-radius: 8px;
  font-size: 0.85em;
}

.debug-info summary {
  cursor: pointer;
  font-weight: 600;
  color: #666;
  margin-bottom: 10px;
}

.debug-info pre {
  background-color: #fff;
  padding: 15px;
  border-radius: 4px;
  overflow-x: auto;
  max-height: 400px;
  overflow-y: auto;
}

.loading {
  text-align: center;
  padding: 50px;
  font-size: 18px;
}

@media (max-width: 768px) {
  .movie-content {
    flex-direction: column;
  }
  
  .movie-poster-container,
  .movie-poster-placeholder {
    width: 100%;
    min-width: auto;
  }
  
  .movie-meta {
    flex-direction: column;
    gap: 10px;
  }
}
</style>