<template>
  <div>
    <nav class="navbar navbar-expand-lg navbar-custom">
      <div class="container-fluid">
        <router-link to="/home" class="navbar-brand">
          <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcShVW-DPqQC9-3Dkh43qw4or6dhdG2AKldxvQ&s" alt="Logo" class="logo">
        </router-link>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav">
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                Películas
              </a>
              <ul class="dropdown-menu">
                <li><router-link class="dropdown-item" to="/categoria?type=movies">Popular</router-link></li>
                <li><router-link class="dropdown-item" to="/categoria?type=movies">En cartelera</router-link></li>
                <li><router-link class="dropdown-item" to="/categoria?type=movies">Próximamente</router-link></li>
                <li><router-link class="dropdown-item" to="/categoria?type=movies">Mejor puntuadas</router-link></li>
              </ul>
            </li>
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                Series
              </a>
              <ul class="dropdown-menu">
                <li><router-link class="dropdown-item" to="/categoria?type=series">Populares</router-link></li>
                <li><router-link class="dropdown-item" to="/categoria?type=series">En emisión</router-link></li>
                <li><router-link class="dropdown-item" to="/categoria?type=series">Mejor puntuadas</router-link></li>
              </ul>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <div class="container mt-4">
      <div v-if="artist">
        <div class="row">
          <div class="col-md-4">
            <img :src="`https://image.tmdb.org/t/p/w500${artist.profile_path}`" :alt="artist.name" class="poster img-fluid">
          </div>
          <div class="col-md-8">
            <h3>{{ artist.name }}</h3>
            <p><strong>Biografía:</strong> {{ artist.biography }}</p>
            <p><strong>Fecha de Nacimiento:</strong> {{ artist.birthday }}</p>
            <p><strong>Lugar de Nacimiento:</strong> {{ artist.place_of_birth }}</p>
          </div>
        </div>

        <h4 class="mt-4">Conocido por:</h4>
        <div class="row">
          <div class="col-md-3" v-for="work in knownFor" :key="work.id">
            <div class="card" @click="goToMovieDetails(work.id)" style="cursor: pointer;">
              <img :src="`https://image.tmdb.org/t/p/w500${work.poster_path}`" :alt="work.title" class="card-img-top" v-if="work.poster_path">
              <div class="card-body">
                <p class="card-text">{{ work.title }}</p>
              </div>
            </div>
          </div>
        </div>

        <h4 class="mt-4">Filmografía:</h4>
        <div class="row">
          <div class="col-md-3" v-for="film in filmography" :key="film.id">
            <div class="card" @click="goToMovieDetails(film.id)" style="cursor: pointer;">
              <img :src="`https://image.tmdb.org/t/p/w500${film.poster_path}`" :alt="film.title" class="card-img-top" v-if="film.poster_path">
              <div class="card-body">
                <p class="card-text">{{ film.title }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-else>
        <p>Cargando detalles...</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { useRoute } from 'vue-router';
export default {
  data() {
    return {
      artist: null,
      knownFor: [],
      filmography: [],
    };
  },
  mounted() {
    const route = useRoute();
    const params = new URLSearchParams(window.location.search);
    const artistId = route.query.artistId;
    if (artistId) {
      this.loadArtistDetails(artistId);
    }
  },
  methods: {
    async loadArtistDetails(artistId) {
      const apiKey = '06524ff7325ce43f515a20c7b39d58a7'; 
      try {
        const response = await axios.get(`https://api.themoviedb.org/3/person/${artistId}?api_key=${apiKey}&language=es`);
        this.artist = response.data;

        const knownForResponse = await axios.get(`https://api.themoviedb.org/3/person/${artistId}/movie_credits?api_key=${apiKey}&language=es`);
        this.knownFor = knownForResponse.data.cast;

        const filmographyResponse = await axios.get(`https://api.themoviedb.org/3/person/${artistId}/combined_credits?api_key=${apiKey}&language=es`);
        this.filmography = filmographyResponse.data.cast;

      } catch (error) {
        console.error('Error al obtener detalles del artista:', error);
      }
    },
    goHome() {
      this.$router.push('/home'); 
    },
    goToMovieDetails(movieId) {
      this.$router.push(`/detalles/${movieId}`); 
    }
  }
};
</script>

<style scoped>
body {
  background-color: #f4f4f4;
  font-family: 'Arial', sans-serif;
}

.navbar-custom {
  background-color: #2c3e50;
}

.navbar-custom a {
  color: #ecf0f1;
}

.navbar-custom a:hover {
  color: #f39c12;
}

.logo {
  height: 40px;
}

h2, h3, h4 {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 20px;
}

.poster {
  width: 100%;
  height: auto;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.card {
  transition: transform 0.3s, box-shadow 0.3s;
}

.card:hover {
  transform: scale(1.05);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
}

.card-text {
  color: #2c3e50;
  text-align: center;
}

.btn-secondary {
  background-color: #34495e;
  border-color: #34495e;
}

.btn-secondary:hover {
  background-color: #2c3e50;
  border-color: #2c3e50;
}

p {
  text-align: center;
  font-size: 18px;
}
</style>
