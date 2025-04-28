<script>
    let location = '';
</script>
  
  <main class="min-h-screen bg-gradient-to-br from-sky-200 to-blue-400 flex flex-col items-center justify-center p-4">
    <div class="bg-white rounded-2xl shadow-lg p-8 w-full max-w-md text-center space-y-6">
      <h1 class="text-3xl font-bold text-gray-800">Weather App</h1>
  
      <input
        type="text"
        bind:value={location}
        placeholder="Enter a location"
        class="w-full p-3 rounded-lg border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-400"
      />
  
      <button
        class="w-full p-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition"
      >
        Search
      </button>
  
      <!-- Weather results section -->
      <div class="mt-6 space-y-2">
        <!-- You can render weather data here later -->
        <div class="text-6xl">🌤️</div>
        <div class="text-2xl font-semibold">--°C</div>
        <div class="text-gray-600">Waiting for location...</div>
      </div>
    </div>
  </main>

<style>
    main {
        width: 20%;
    }
</style>