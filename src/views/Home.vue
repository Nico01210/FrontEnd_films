<template>
    <h1>NETFLOX</h1>
    
    <H2> Bienvenue sur votre site de film en streaming</H2>
  <div>
    <SearchBar @search="handleSearch" />
    <div class="movie-list">
      <MovieCard v-for="movie in movies" :key="movie.id" :movie="movie" />
    </div>
    <div v-if="movies.length === 0 && searchQuery" class="no-results">
      <p>Aucun film trouvé pour "{{ searchQuery }}"</p>
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
      searchQuery: ''
    };
  },
  methods: {
    async fetchMovies(page = 1, search = '') {
      try {
        let url = `/movies?page=${page}`;
        if (search) {
          url += `&title=${encodeURIComponent(search)}`;
        }
        console.log('URL de recherche:', url);
        const res = await api.get(url);
        console.log('Résultats de recherche:', res.data);
        this.movies = res.data.member || res.data['hydra:member'] || [];
        this.totalItems = res.data.totalItems || res.data['hydra:totalItems'] || 0;
        this.page = page;
      } catch(err) {
        console.error('Erreur de recherche:', err);
        this.movies = [];
        this.totalItems = 0;
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
.movie-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
  max-width: 1400px;
  margin: 0 auto;
}

.no-results {
  text-align: center;
  padding: 50px;
  color: white;
  font-size: 20px;
}

.no-results p {
  margin: 0;
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
