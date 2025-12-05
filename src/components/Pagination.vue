<template>
  <div class="pagination" v-if="totalPages > 1">
    <button 
      @click="changePage(currentPage - 1)" 
      :disabled="currentPage === 1"
      class="page-button"
    >
      ← Précédent
    </button>
    
    <span class="page-info">Page {{ currentPage }} sur {{ totalPages }}</span>
    
    <button 
      @click="changePage(currentPage + 1)" 
      :disabled="currentPage === totalPages"
      class="page-button"
    >
      Suivant →
    </button>
  </div>
</template>

<script>
export default {
  props: {
    total: {
      type: Number,
      required: true
    },
    currentPage: {
      type: Number,
      default: 1
    },
    perPage: {
      type: Number,
      default: 20
    }
  },
  computed: {
    totalPages() {
      return Math.ceil(this.total / this.perPage);
    }
  },
  methods: {
    changePage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.$emit('changePage', page);
      }
    }
  }
};
</script>

<style scoped>
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  padding: 30px 20px;
}

.page-button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.page-button:hover:not(:disabled) {
  background-color: #359268;
}

.page-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.page-info {
  font-size: 16px;
  color: #555;
  font-weight: 500;
}
</style>