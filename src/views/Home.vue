<template>
    <h1></h1>
  <div>
    <SearchBar @search="fetchMovies" />
    <div class="movie-list">
      <MovieCard v-for="movie in movies" :key="movie.id" :movie="movie" />
    </div>
    <Pagination :total="totalItems" :currentPage="page" @changePage="fetchMovies" />
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
      page: 1
    };
  },
  methods: {
    async fetchMovies(page = 1) {
      try {
        const res = await api.get(`/movies?page=${page}`);
        console.log(res.data);
        this.movies = res.data.member;
        this.totalItems = res.data.totalItems;
        this.page = page;
      } catch(err) {
        console.error(err);
      }
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

@media (max-width: 768px) {
  .movie-list {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 15px;
    padding: 15px;
  }
}
</style>
