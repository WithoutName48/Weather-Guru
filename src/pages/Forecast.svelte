<script>
    import Template from "../components/Template.svelte";

    import { tick } from "svelte";
    import Chart from 'chart.js/auto';
  
    let location = "";
    let latitude = null;
    let longitude = null;
    let weatherData = null;
    let chart;

    // Function to get the coordinates of the location
    async function getCoordinates() {
      const res = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(location)}&count=1`);
      const data = await res.json();
      if (!data.results || data.results.length === 0) {
        alert("Location not found");
        return;
      }
      latitude = data.results[0].latitude;
      longitude = data.results[0].longitude;
    }
  
    // Function to fetch forecast data
    async function fetchData() {
      await getCoordinates();
      if (!latitude || !longitude) return;
  
      const url = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&daily=temperature_2m_max,temperature_2m_min&timezone=auto&forecast_days=16`;
  
      const res = await fetch(url);
      weatherData = await res.json();

      await tick(); // wait for DOM update

      // After fetching, draw chart
      drawChart();
    }
  
    // Function to draw the forecast chart
    function drawChart() {
      if (!weatherData) return;
      
      const canvasWeather = document.getElementById('weatherChart');
      // @ts-ignore
      const ctx = canvasWeather.getContext('2d');
  
      if (chart) {
        chart.destroy();
      }
 
      const labels = weatherData.daily.time;
      const dataset1 = weatherData.daily.temperature_2m_max;
      const dataset2 = weatherData.daily.temperature_2m_min;
      const label1 = "Max Temp (°C)";
      const label2 = "Min Temp (°C)";
  
      chart = new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              label: label1,
              data: dataset1,
              borderColor: 'blue',
              backgroundColor: 'lightblue',
              fill: false,
            },
            {
              label: label2,
              data: dataset2,
              borderColor: 'red',
              backgroundColor: 'pink',
              fill: false,
            }
          ]
        },
        options: {
          responsive: true,
          scales: {
            y: {
              beginAtZero: true,
            }
          }
        }
      });
    }
  </script>
  
  <div id="index">
    <Template />
  
    <div id="index_content" class="max-h-full">
        <div class="min-w-[300px] max-w-[90%] max-h-full bg-white bg-opacity-80 p-10 rounded-2xl shadow-lg text-center space-y-6">
            <div class="w-full flex flex-col items-center gap-4 max-h-full">
                <div class="flex flex-col sm:flex-row gap-2 w-full max-w-2xl">
                <input
                    type="text"
                    bind:value={location}
                    placeholder="Enter location"
                    class="p-3 w-full border rounded"
                />
                </div>
        
                <button on:click={fetchData} class="w-full p-3 !bg-blue-500 text-white rounded-lg hover:bg-blue-600 transitio">
                Fetch Forecast
                </button>
        
                {#if weatherData}
                <div class="w-full max-w-4xl">
                    <canvas id="weatherChart"></canvas>
                </div>
        
                <div class="mt-6 w-full overflow-auto max-h-[400px]">
                    <table class="min-w-full bg-white rounded-lg w-full">
                    <thead class="bg-gray-200">
                        <tr>
                        <th class="py-2 px-4">Date</th>
                        <th class="py-2 px-4">Max Temp (°C)</th>
                        <th class="py-2 px-4">Min Temp (°C)</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#each weatherData.daily.time as time, index}
                        <tr class="border-t">
                            <td class="py-2 px-4 text-center">{time}</td>
                            <td class="py-2 px-4 text-center">{weatherData.daily.temperature_2m_max[index]}</td>
                            <td class="py-2 px-4 text-center">{weatherData.daily.temperature_2m_min[index]}</td>
                        </tr>
                        {/each}
                    </tbody>
                    </table>
                </div>
                {/if}
            </div>
        </div>
    </div>
  </div>
  
  <style>
    #index {
      width: 100vw;
      height: 100vh;
      position: static;
      background-image: url('https://preview.redd.it/nduksi52zrs41.jpg?auto=webp&s=0760039b9b3ebdb699ef6569ee313b787f27375c');
      background-size: cover;
      background-position: center center;
      display: flex;
      flex-direction: row;
    }
  
    #index_content {
      width: 100%;
      height: fit-content;
      padding: 2rem;

      display: flex;
      justify-content: center;
      align-items: center;
    }
  </style>
  