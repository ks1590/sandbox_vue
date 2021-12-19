<template>
  <div class="home">
    <Hero />

    <div class="container movies">
      <div id="movie-greid" class="movie-grid">
        <div class="movie" v-for="movie in movies" :key="movie.id">
          <div class="movie-img">
            <img :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`" alt="">
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">{{ movie.title.slice(0, 25) }}
              <span v-if="movie.title.length > 25">...</span>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      movies: [],
    }
  },
  async fetch() {
    await this.getMovies();
  },
  methods: {
    async getMovies() {
      const data = axios.get(
        'https://api.themoviedb.org/3/movie/now_playing?api_key=37ed43a4f8eaa2abd75f9283692947bc&language=en-US&page=1'  
      );
      const result = await data;
      result.data.results.forEach(movie => {
        this.movies.push(movie);
      });
      console.log(result.data.results);
    },
  },
}
</script>
