<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  let movieList = writable([]);

  const formatRuntime = (runtime) => {
    const hours = Math.floor(runtime / 60);
    const minutes = runtime % 60;
    return `${hours}h ${minutes}min`;
  };

  const getLanguageFullName = (languageCode) => {
    const languageMap = {
      'en': 'English', 'es': 'Spanish', 'fr': 'French', 'de': 'German',
      'it': 'Italian', 'ja': 'Japanese', 'ko': 'Korean', 'zh': 'Chinese',
      'ru': 'Russian', 'pt': 'Portuguese', 'ar': 'Arabic', 'hi': 'Hindi',
      'sv': 'Swedish', 'nl': 'Dutch', 'tr': 'Turkish', 'no': 'Norwegian',
      'da': 'Danish', 'fi': 'Finnish', 'pl': 'Polish', 'hu': 'Hungarian',
      'cs': 'Czech', 'ro': 'Romanian', 'el': 'Greek', 'he': 'Hebrew',
      'th': 'Thai', 'id': 'Indonesian', 'vi': 'Vietnamese',
      // ... other languages
    };
    return languageMap[languageCode] || languageCode;
  };

  const getMoviesWithDetails = async (page) => {
    try {
      const res = await fetch(`https://api.themoviedb.org/3/discover/movie?api_key=d17f4fce1773d642f23563b737b4f7b3&page=${page}`);
      const json = await res.json();
      return json.results;
    } catch (error) {
      console.error("Error fetching movies:", error);
      return [];
    }
  };

  onMount(async () => {
    const fetchMovies = async () => {

      const moviesPage1 = await getMoviesWithDetails(1);
      const moviesPage2 = await getMoviesWithDetails(2);

      const combinedMovies = [...moviesPage1, ...moviesPage2];

      const moviesWithDetailsPromises = combinedMovies.map(async (movie) => {
        const detailsRes = await fetch(`https://api.themoviedb.org/3/movie/${movie.id}?api_key=d17f4fce1773d642f23563b737b4f7b3`);
        const detailsJson = await detailsRes.json();
        const creditsRes = await fetch(`https://api.themoviedb.org/3/movie/${movie.id}/credits?api_key=d17f4fce1773d642f23563b737b4f7b3`);
        const creditsJson = await creditsRes.json();
        const videosRes = await fetch(`https://api.themoviedb.org/3/movie/${movie.id}/videos?api_key=d17f4fce1773d642f23563b737b4f7b3`);
        const videosJson = await videosRes.json();

        const director = creditsJson.crew.find(member => member.job === 'Director');
        const cast = creditsJson.cast.map(actor => actor.name).join(', ');
        const genres = detailsJson.genres.map(genre => genre.name).join(', ');
        const trailer = videosJson.results.find(video => video.type === 'Trailer' && video.site === 'YouTube');

        return {
          ...movie,
          director: director ? director.name : 'N/A',
          cast,
          genres,
          runtime: formatRuntime(detailsJson.runtime),
          vote_average: `${detailsJson.vote_average}/10`,
          original_language: getLanguageFullName(detailsJson.original_language),
          original_title: detailsJson.original_title,
          budget: detailsJson.budget ? `$${detailsJson.budget.toLocaleString()}` : 'N/A',
          trailer: trailer ? `https://www.youtube.com/watch?v=${trailer.key}` : null,
        };
      });

      const moviesWithDetails = await Promise.all(moviesWithDetailsPromises);
      movieList.set(moviesWithDetails);
    };

    fetchMovies();
  });
</script>

<div>
  <header class="App-header">
    <h1>Scroll down to view popular movies</h1>
    <p>( made with Svelte )</p>
  </header>
  <div class="movie-list">
    {#each $movieList as movie (movie.id)}
      <div class="movie-card" key={movie.id}>
        <img class="movie-card-image"
             src={`https://image.tmdb.org/t/p/w500${movie.poster_path}`}
             alt={movie.title}
        />
        <div class="movie-card-content">
          <h2>{movie.title}</h2>
          <p><strong>Original Title:</strong> {movie.original_title}</p>
          <p><strong>Release Date:</strong> {movie.release_date}</p>
          <p><strong>Genres:</strong> {movie.genres}</p>
          <p><strong>Duration:</strong> {movie.runtime}</p>
          <p><strong>Rating:</strong> {movie.vote_average}</p>
          <p><strong>Summary:</strong> {movie.overview}</p>
          <p><strong>Adult:</strong> {movie.adult ? 'For adults' : 'Not exclusively for adults'}</p>
          <p><strong>Director:</strong> {movie.director}</p>
          <p><strong>Cast:</strong> {movie.cast}</p>
          <p><strong>Original Language:</strong> {movie.original_language}</p>
          <p><strong>Budget:</strong> {movie.budget}</p>
          {#if movie.trailer}
            <p>
              <a href={movie.trailer} target="_blank" rel="noopener noreferrer">Watch Trailer</a>
            </p>
          {/if}
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  .movie-card {
    background-color: #1e1e1e;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.7);
    color: #ffffff;
    margin: 20px 0;
    padding: 20px;
    width: 70%;
    max-width: 600px;
  }

  .movie-list {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 20px;
  }



  .movie-card-image {
    width: 50%;
    height: auto;
    border-radius: 8px;
    display: block;
    margin-left: auto;
    margin-right: auto;
  }


  .movie-card-content {
    padding: 16px;
  }

  .movie-card-content h2 {
    text-align: center;
    margin-top: 1em;
    margin-bottom: 2em;
  }

  @media (max-width: 768px) {
    .movie-card {
      flex: 0 1 calc(100% - 20px);
      max-width: calc(100% - 20px);
    }
  }

  .App-header {
    text-align: center;
    background-color: #121212;
    color: white;
    padding: 20px;
    font-size: calc(10px + 2vmin);
  }
</style>
