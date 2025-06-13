<script>
  import { tick } from "svelte";
  import Chart from "chart.js/auto";

  let location = "";
  let startDate = "";
  let endDate = "";
  let latitude = null;
  let longitude = null;
  let weatherData = null;
  let canvasWeather;
  let chart;

  const today = new Date();
  const yesterday = new Date(today);
  yesterday.setDate(today.getDate() - 1);
  const maxDate = yesterday.toISOString().split("T")[0];

  // Reactive validation for incorrect date range
  $: incorrectDateRange = startDate !== "" && endDate !== "" && startDate > endDate;

  async function getCoordinates() {
    try {
      const res = await fetch(
        `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(location)}&count=1`
      );
      const data = await res.json();
      if (!data.results || data.results.length === 0) {
        alert("Location not found");
        return;
      }
      latitude = data.results[0].latitude;
      longitude = data.results[0].longitude;
    } catch (err) {
      console.error("Error fetching coordinates:", err);
      alert("Failed to get coordinates.");
    }
  }

async function fetchData() {
  if (incorrectDateRange) return;

  await getCoordinates();
  if (!latitude || !longitude) return;

  const url = `https://archive-api.open-meteo.com/v1/archive?latitude=${latitude}&longitude=${longitude}&start_date=${startDate}&end_date=${endDate}&daily=temperature_2m_max,temperature_2m_min`;

  try {
    const res = await fetch(url);
    weatherData = await res.json();

    if (!weatherData?.daily?.temperature_2m_max?.length) {
      alert("No weather data available.");
      return;
    }

    await tick(); // ⬅ Wait for DOM to update (canvas bind)
    drawChart();
  } catch (err) {
    console.error("Error fetching weather data:", err);
    alert("Failed to fetch weather data.");
  }
}


  function drawChart() {
    if (!canvasWeather || !weatherData) return;

    const ctx = canvasWeather.getContext("2d");

    if (chart) {
      chart.destroy();
    }

    chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: weatherData.daily.time.map((val)=>{
            return val.replace("T", " ");
        }),
        datasets: [
          {
            label: "Max Temp (°C)",
            data: weatherData.daily.temperature_2m_max,
            borderColor: "blue",
            backgroundColor: "lightblue",
            fill: false,
          },
          {
            label: "Min Temp (°C)",
            data: weatherData.daily.temperature_2m_min,
            borderColor: "red",
            backgroundColor: "pink",
            fill: false,
          },
        ],
      },
      options: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true,
          },
        },
      },
    });
  }
</script>

<div id="hw_content">
  <div class="index_content bg-white bg-opacity-80 p-10 rounded-2xl shadow-lg min-w-2xl max-w-full text-center space-y-6">
    <div class="w-full flex flex-col items-center gap-4">
      <div class="flex flex-wrap flex-col gap-2 w-full">
        <div class="flex justify-center">
          <label class="w-full">
            <h1 class="index_logo text-lg-3xl text-xs-xl font-bold text-gray-800">Location</h1><br />
            <input
              type="text"
              bind:value={location}
              placeholder="Type..."
              class="p-3 w-full rounded border border-gray-400 text-gray-400 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
            />
          </label>
        </div>

        <div class="w-full flex justify-around gap-5 text-gray-600">
          <label class="w-full">
            <p class="font-bold">Start date:</p>
            <input
              type="date"
              bind:value={startDate}
              min="1940-01-01"
              max={maxDate}
              class="p-3 w-full border rounded border-gray-400 text-gray-400 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
            />
          </label>
          <label class="w-full">
            <p class="font-bold">End date:</p>
            <input
              type="date"
              bind:value={endDate}
              min="1940-01-01"
              max={maxDate}
              class="p-3 w-full border rounded border-gray-400 text-gray-400 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
            />
          </label>
        </div>

        {#if incorrectDateRange}
          <p class="text-right text-red-600">Start date must be before end date</p>
        {/if}
      </div>

      <button
        on:click={fetchData}
        class="w-full p-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition"
      >
        Fetch historical data
      </button>

      {#if weatherData}
        <div class="w-full max-w-4xl bg-white p-5">
          <canvas
            id="weatherChart"
            height="400"
            width="800"
            bind:this={canvasWeather}
          ></canvas>
        </div>

        <div class="mt-6 w-full overflow-auto max-h-[400px]">
          <table class="index_content min-w-full text-gray-400 rounded-lg w-full">
            <thead class="bg-blue-400 text-white">
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
                  <td class="py-2 px-4 text-center">
                    {weatherData.daily.temperature_2m_max[index]}
                  </td>
                  <td class="py-2 px-4 text-center">
                    {weatherData.daily.temperature_2m_min[index]}
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {/if}
    </div>
  </div>
</div>

<style>
  #hw_content {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 3%;
    height: 100%;
  }
</style>
