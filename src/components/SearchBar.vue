<template>
  <div class="search-bar">
    <input 
      type="text" 
      v-model="searchQuery" 
      @input="onSearch" 
      placeholder="Rechercher un film..."
      class="search-input"
    />
    <button @click="onSearch" class="search-button">🔍</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: ''
    };
  },
  methods: {
    onSearch() {
      this.$emit('search', this.searchQuery);
    }
  }
};
</script>

<style scoped>
.search-bar {
  display: flex;
  gap: 10px;
  padding: 20px;
  max-width: 600px;
  margin: 0 auto;
}

.search-input {
  flex: 1;
  padding: 12px 20px;
  border: 2px solid #ddd;
  border-radius: 25px;
  font-size: 16px;
  outline: none;
  transition: border-color 0.3s;
}

.search-input:focus {
  border-color: #42b983;
}

.search-button {
  padding: 12px 24px;
  background-color: #324dc4;
  color: white;
  border: none;
  border-radius: 25px;
  font-size: 18px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.search-button:hover {
  background-color: #101e5c;
}
</style>