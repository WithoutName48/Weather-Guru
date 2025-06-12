<script>
    let location = '';
    let temperature = null;
    let weatherCode = null;
  
    const weatherIcons = {
      0: "☀️", // Clear sky
      1: "🌤️", // Mainly clear
      2: "⛅", // Partly cloudy
      3: "☁️", // Overcast
      45: "🌫️", // Fog
      48: "🌫️", // Depositing rime fog
      51: "🌦️", // Light drizzle
      61: "🌧️", // Rain
      71: "🌨️", // Snow
      95: "⛈️", // Thunderstorm
    };
  
    async function fetchWeather() {
      try {
        const geoRes = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(location)}&count=1`);
        const geoData = await geoRes.json();
  
        if (!geoData.results || geoData.results.length === 0) {
          alert('Location not found');
          return;
        }
  
        const { latitude, longitude } = geoData.results[0];
  
        const weatherRes = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true`);
        const weatherData = await weatherRes.json();
  
        temperature = weatherData.current_weather.temperature;
        weatherCode = weatherData.current_weather.weathercode;
      } catch (error) {
        console.error('Error fetching weather:', error);
      }
    }
  
    function handleKeyDown(event) {
      if (event.key === 'Enter') {
        fetchWeather();
      }
    }
  </script>
  
<div class="container w-full max-w-full min-h-screen flex-grow h-fit bg-gradient-to-br from-sky-200/40 to-blue-400/40 backdrop-blur-md flex flex-col items-center justify-center p-4">
    <div class="bg-white rounded-2xl shadow-lg p-8 w-full max-w-md text-center space-y-8" style="height: fit-content;">
      <h1 class="text-lg-3xl text-xs-xl font-bold text-gray-800">
        Weather<span class="text-blue-800">Guru</span>
      </h1>
  
      <input
        type="text"
        bind:value={location}
        placeholder="Enter a location and press Enter"
        on:keydown={handleKeyDown}
        class="w-full p-3 rounded-lg border border-gray-300 text-gray-300 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
      />
  
      <div class="flex flex-col space-y-3">
        <a href="/#/forecast" class="w-full p-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition">
          Forecast Weather (16 days)
        </a>
  
        <a href="/#/history" class="w-full p-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition">
          Historical Weather (since 1940)
        </a>
  
        <a href="/#/air-quality" class="w-full p-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition">
          Air Quality Forecast (7 days)
        </a>
  
        <a href="/#/marine" class="w-full p-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition">
          Marine Weather Forecast (16 days)
        </a>
      </div>
  
      <div class="mt-8 space-y-2">
        <div class="text-6xl">
          {#if weatherCode !== null}
            {weatherIcons[weatherCode] || "❓"}
          {:else}
            🌤️
          {/if}
        </div>
        <div class="text-2xl text-blue-400 font-semibold">
          {#if temperature !== null}
            {temperature}°C
          {:else}
            --°C
          {/if}
        </div>
        <div class="text-gray-600">
          {#if temperature !== null}
            Showing current weather
          {:else}
            Waiting for location...
          {/if}
        </div>
      </div>
    </div>
  </div>