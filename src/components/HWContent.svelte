<script>
  import Template from "../components/Template.svelte";

  import Chart from "chart.js/auto";

  let location = $state("");
  let startDate = $state("");
  let endDate = $state("");
  let latitude = null;
  let longitude = null;
  let weatherData = null;
  let fetchedData = $state(false);
  let canvasWeather;
  let chart;

  const today = new Date();
  const yesterday = new Date(today);
  yesterday.setDate(today.getDate() - 1);

  const maxDate = yesterday.toISOString().split("T")[0];

  const incorrectDateRange = $derived(
    startDate !== "" && endDate !== "" && startDate > endDate
  );

  // Function to get the coordinates of the location
  async function getCoordinates() {
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
  }

  // Function to fetch forecast data
  async function fetchData() {
    if (incorrectDateRange) return;
    await getCoordinates();
    if (!latitude || !longitude) return;

    const url = `https://archive-api.open-meteo.com/v1/archive?latitude=${latitude}&longitude=${longitude}&start_date=${startDate}&end_date=${endDate}&daily=temperature_2m_max,temperature_2m_min`;

    const res = await fetch(url);
    weatherData = await res.json();

    $: if (weatherData && canvasWeather) {
      drawChart();
    }
  }

  // Function to draw the forecast chart
  function drawChart() {
    const ctx = canvasWeather.getContext("2d");

    if (chart) {
      chart.destroy();
    }

    const labels = weatherData.daily.time;
    const dataset1 = weatherData.daily.temperature_2m_max;
    const dataset2 = weatherData.daily.temperature_2m_min;
    const label1 = "Max Temp (°C)";
    const label2 = "Min Temp (°C)";

    chart = new Chart(ctx, {
      type: "line",
      data: {
        labels,
        datasets: [
          {
            label: label1,
            data: dataset1,
            borderColor: "blue",
            backgroundColor: "lightblue",
            fill: false,
          },
          {
            label: label2,
            data: dataset2,
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

  <div
    id="hw_content">
    <div class="bg-white bg-opacity-80 p-10 rounded-2xl shadow-lg max-w-2xl text-center space-y-6">
      <div class="w-full flex flex-col items-center gap-4">
        <div class="flex flex-wrap flex-col gap-2 w-full max-w-2xl">
          <div class="flex justify-center">
            <label class="w-full">
              <h1 class="text-lg-3xl text-xs-xl font-bold text-gray-800">Location</h1><br>
              <input
                type="text"
                bind:value={location}
                placeholder="Type..."
                class="p-3 w-full rounded border border-gray-300 text-gray-300 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
              />
            </label>
          </div>
          <div class="w-full flex justify-center gap-5 text-gray-600">
            <label>
              <p class="font-bold">Start date:</p>
              <input
                type="date"
                bind:value={startDate}
                min="1940-01-01"
                max={maxDate}
                class="p-3 border rounded border-gray-300 text-gray-300 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
              />
            </label>
            <label>
            <p class="font-bold">End date:</p>
              <input
                type="date"
                bind:value={endDate}
                min="1940-01-01"
                max={maxDate}
                class="p-3 border rounded border-gray-300 text-gray-300 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
              />
            </label>
          </div>
          {#if incorrectDateRange}
            <p class="text-right text-red-600">
              start date must be before end date
            </p>
          {/if}
        </div>

        <button
          onclick={fetchData}
          class="w-full p-3 !bg-blue-500 text-white rounded-lg hover:bg-blue-600 transitio"
        >
          Fetch historical data
        </button>

        {#if weatherData}
          <div class="w-full max-w-4xl">
            <canvas
              id="weatherChart"
              height="400"
              width="800"
              bind:this={canvasWeather}
            ></canvas>
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
                    <td class="py-2 px-4 text-center"
                      >{weatherData.daily.temperature_2m_max[index]}</td
                    >
                    <td class="py-2 px-4 text-center"
                      >{weatherData.daily.temperature_2m_min[index]}</td
                    >
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
