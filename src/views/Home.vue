<template>
  <div class="home">
    <h2 class="welcome"> Bienvenue sur votre site de film en streaming</h2>
    <SearchBar @search="handleSearch" />
    <div class="movie-list">
      <MovieCard v-for="movie in movies" :key="movie.id" :movie="movie" />
    </div>
    <div v-if="movies.length === 0 && !loading" class="no-results">
      <p v-if="searchQuery">Aucun film trouvé pour "{{ searchQuery }}"</p>
      <p v-else-if="error">{{ error }}</p>
      <p v-else>Aucun film disponible</p>
    </div>
    <div v-if="loading" class="loading">
      <p>Chargement...</p>
    </div>
    <Pagination :total="totalItems" :currentPage="page" @changePage="changePage" />
  </div>
</template>

<script>
import MovieCard from '../components/MovieCard.vue';
import Pagination from '../components/Pagination.vue';
import SearchBar from '../components/SearchBar.vue';
import api from '../api/axios.js';

export default {
  components: { MovieCard, Pagination, SearchBar },
  data() {
    return {
      movies: [],
      totalItems: 0,
      page: 1,
      searchQuery: '',
      loading: false,
      error: null
    };
  },
  methods: {
    async fetchMovies(page = 1, search = '') {
      this.loading = true;
      this.error = null;
      try {
        let url = `/movies?page=${page}`;
        if (search) {
          url += `&title=${encodeURIComponent(search)}`;
        }
        console.log('URL de recherche:', url);
        console.log('Base URL:', api.defaults.baseURL);
        const res = await api.get(url);
        console.log('Résultats de recherche:', res.data);
        this.movies = res.data.member || res.data['hydra:member'] || [];
        this.totalItems = res.data.totalItems || res.data['hydra:totalItems'] || 0;
        this.page = page;
      } catch(err) {
        console.error('Erreur de recherche:', err);
        console.error('Détails:', err.response?.data);
        console.error('Status:', err.response?.status);
        this.error = 'Erreur lors du chargement des films. Vérifiez que votre serveur backend est démarré.';
        this.movies = [];
        this.totalItems = 0;
      } finally {
        this.loading = false;
      }
    },
    handleSearch(query) {
      console.log('Recherche:', query);
      this.searchQuery = query;
      this.page = 1;
      this.fetchMovies(1, query);
    },
    changePage(page) {
      this.fetchMovies(page, this.searchQuery);
    }
  },
  mounted() {
    this.fetchMovies();
  }
};
</script>

<style scoped>
.home {
  padding: 20px 0;
}

.welcome {
  text-align: center;
  color: white;
  font-size: 24px;
  margin: 20px 0;
}

.movie-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
  max-width: 1400px;
  margin: 0 auto;
}

.no-results,
.loading {
  text-align: center;
  padding: 50px;
  color: white;
  font-size: 20px;
}

.no-results p,
.loading p {
  margin: 0;
}

.error {
  color: #ff6b6b;
}

.page-info {
  font-size: 16px;
  color: white;
  font-weight: 500;
}

@media (max-width: 768px) {
  .movie-list {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 15px;
    padding: 15px;
  }
}
</style>