<template>
  <div>
    <nav class="navbar navbar-expand-lg navbar-custom">
      <div class="container-fluid">
        <a class="navbar-brand" href="/">
          <img src="https://w7.pngwing.com/pngs/162/500/png-transparent-popcorn-popcorn-box-food.png" alt="Logo" class="logo" />
        </a>
        <button
          class="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#navbarNav"
          aria-controls="navbarNav"
          aria-expanded="false"
          aria-label="Toggle navigation"
        >
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav me-auto">
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                Películas
              </a>
              <ul class="dropdown-menu">
                <li><a class="dropdown-item" href="/categoria?categoryId=28">Popular</a></li>
                <li><a class="dropdown-item" href="/categoria?categoryId=29">En cartelera</a></li>
                <li><a class="dropdown-item" href="/categoria?categoryId=30">Próximamente</a></li>
                <li><a class="dropdown-item" href="/categoria?categoryId=31">Mejor puntuadas</a></li>
              </ul>
            </li>
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                Series
              </a>
              <ul class="dropdown-menu">
                <li><a class="dropdown-item" href="/categoria?categoryId=32">Popular</a></li>
                <li><a class="dropdown-item" href="/categoria?categoryId=33">En emisión</a></li>
                <li><a class="dropdown-item" href="/categoria?categoryId=34">Mejor puntuadas</a></li>
              </ul>
            </li>
          </ul>
          <ul class="navbar-nav ms-auto">
            <li class="nav-item">
              <span class="nav-link">{{ username ? `Hola, ${username}` : 'Usuario' }}</span>
            </li>
            <li class="nav-item">
              <button class="btn btn-danger" @click="logout">Cerrar sesión</button>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <input v-model="searchQuery" placeholder="Buscar..." @input="searchContent" />
    <div v-if="searchResults.length">
      <ul>
        <li v-for="result in searchResults" :key="result.id" @click="goToDetails(result)">
          <span>{{ result.name || result.title }}</span>
        </li>
      </ul>
    </div>

    <div class="container mt-4">
      <h2>Tendencias</h2>
      <div class="movie-section" v-if="movies.trending.length">
        <div v-for="movie in movies.trending" :key="movie.id" @click="goToDetails(movie)">
          <img :src="'https://image.tmdb.org/t/p/w500' + movie.poster_path" :alt="movie.title" class="poster" />
          <p class="movie-title">{{ movie.title }}</p>
        </div>
      </div>
      <button v-if="trendingPage < totalPages.trending" @click="loadMore('trending')" class="btn btn-primary">Cargar más</button>

      <h2>Lo más popular</h2>
      <div class="movie-section" v-if="movies.popular.length">
        <div v-for="movie in movies.popular" :key="movie.id" @click="goToDetails(movie)">
          <img :src="'https://image.tmdb.org/t/p/w500' + movie.poster_path" :alt="movie.title" class="poster" />
          <p class="movie-title">{{ movie.title }}</p>
        </div>
      </div>
      <button v-if="popularPage < totalPages.popular" @click="loadMore('popular')" class="btn btn-primary">Cargar más</button>

      <h2>Ver gratis</h2>
      <div class="movie-section" v-if="movies.free.length">
        <div v-for="movie in movies.free" :key="movie.id" @click="goToDetails(movie)">
          <img :src="'https://image.tmdb.org/t/p/w500' + movie.poster_path" :alt="movie.title" class="poster" />
          <p class="movie-title">{{ movie.title }}</p>
        </div>
      </div>
      <button v-if="freePage < totalPages.free" @click="loadMore('free')" class="btn btn-primary">Cargar más</button>

      <h2>Series o TV</h2>
      <div class="movie-section" v-if="movies.series.length">
        <div v-for="series in movies.series" :key="series.id" @click="goToSeriesDetails(series.id)">
          <img :src="'https://image.tmdb.org/t/p/w500' + series.poster_path" :alt="series.name" class="poster" />
          <p class="movie-title">{{ series.name }}</p>
        </div>
      </div>
      <button v-if="seriesPage < totalPages.series" @click="loadMore('series')" class="btn btn-primary">Cargar más</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';

const username = ref('');
const searchQuery = ref('');
const searchResults = ref([]);
const movies = ref({
  trending: [],
  popular: [],
  free: [],
  series: []
});
const trendingPage = ref(1);
const popularPage = ref(1);
const freePage = ref(1);
const seriesPage = ref(1);
const totalPages = ref({
  trending: 1,
  popular: 1,
  free: 1,
  series: 1
});

const router = useRouter();

onMounted(() => {
  username.value = localStorage.getItem('username');
  fetchMovies('trending', '/trending/movie/day', trendingPage.value);
  fetchMovies('popular', '/movie/popular', popularPage.value);
  fetchMovies('free', '/movie/top_rated', freePage.value);
  fetchMovies('series', '/tv/popular', seriesPage.value);
});

const fetchMovies = (category, endpoint, page) => {
  const apiKey = '06524ff7325ce43f515a20c7b39d58a7';
  axios
    .get(`https://api.themoviedb.org/3${endpoint}?api_key=${apiKey}&language=es&page=${page}`)
    .then((response) => {
      movies.value[category] = movies.value[category].concat(response.data.results);
      totalPages.value[category] = response.data.total_pages;
    })
    .catch((error) => {
      console.error('Error al obtener datos de TMDb:', error);
    });
};

const loadMore = (category) => {
  if (category === 'trending') {
    trendingPage.value++;
    fetchMovies('trending', '/trending/movie/day', trendingPage.value);
  } else if (category === 'popular') {
    popularPage.value++;
    fetchMovies('popular', '/movie/popular', popularPage.value);
  } else if (category === 'free') {
    freePage.value++;
    fetchMovies('free', '/movie/top_rated', freePage.value);
  } else if (category === 'series') {
    seriesPage.value++;
    fetchMovies('series', '/tv/popular', seriesPage.value);
  }
};

const searchContent = () => {
  if (searchQuery.value.length > 2) {
    axios
      .get(`https://api.themoviedb.org/3/search/multi?api_key=06524ff7325ce43f515a20c7b39d58a7&query=${searchQuery.value}`)
      .then((response) => {
        searchResults.value = response.data.results.filter(
          result => result.media_type === 'movie' || result.media_type === 'tv'
        );
      });
  } else {
    searchResults.value = [];
  }
};

const goToDetails = (result) => {
  if (result.media_type === 'movie') {
    router.push(`/detalles/${result.id}`);
  } else if (result.media_type === 'tv') {
    router.push(`/detallesserie/${result.id}`);
  }
};

const goToSeriesDetails = (seriesId) => {
  router.push({ name: 'DetallesSeries', params: { seriesId } });
};

const logout = () => {
  localStorage.removeItem('sessionId');
  router.push({ name: 'Login' });
};
</script>

<style scoped>
.navbar-custom {
  background-color: #2c3e50;
}

.navbar-custom a {
  color: #ecf0f1;
  margin-right: 20px;
}

.navbar-custom a:hover {
  color: #f39c12;
}

.logo {
  height: 40px;
}

.dropdown-menu .dropdown-item {
  color: #000;
}

.dropdown-menu .dropdown-item:hover {
  background-color: #f39c12;
  color: #fff;
}

input {
  padding: 8px;
  margin-bottom: 16px;
  width: 100%;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 8px;
  background: #f0f0f0;
  margin-bottom: 4px;
  cursor: pointer;
}

li:hover {
  background: #e0e0e0;
}

h2 {
  font-size: 1.8rem;
  color: #f39c12;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.6);
  margin-bottom: 20px;
}

.poster {
  width: 150px;
  height: 225px;
  object-fit: cover;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.movie-section {
  display: flex;
  overflow-x: auto;
  gap: 15px;
  padding: 10px;
  scrollbar-width: thin;
  scrollbar-color: #f39c12 #2c3e50;
}

.movie-section::-webkit-scrollbar {
  height: 8px;
}

.movie-section::-webkit-scrollbar-thumb {
  background-color: #f39c12;
  border-radius: 10px;
}

.movie-section div {
  position: relative;
  transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
}

.movie-section div:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.movie-title {
  text-align: center;
  font-size: 1rem;
  color: #000;
  margin-top: 10px;
}

.container {
  padding: 20px;
}

body {
  background: linear-gradient(135deg, #2c3e50, #34495e);
}
</style>
