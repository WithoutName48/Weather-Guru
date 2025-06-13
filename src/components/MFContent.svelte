<script>
  import Template from "../components/Template.svelte";

  import { tick } from "svelte";
  import Chart from "chart.js/auto";

  let location = "";
  let latitude = null;
  let longitude = null;
  let marineData = null;
  let chart;

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
    await getCoordinates();
    if (!latitude || !longitude) return;

    const url = `https://marine-api.open-meteo.com/v1/marine?latitude=${latitude}&longitude=${longitude}&hourly=wave_height,ocean_current_velocity&forecast_days=16`;

    const res = await fetch(url);
    marineData = await res.json();

    await tick(); // wait for DOM update

    // After fetching, draw chart
    drawChart();
  }

  // Function to draw the forecast chart
  function drawChart() {
    if (!marineData) return;

    const canvasMarine = document.getElementById("marineChart");
    // @ts-ignore
    const ctx = canvasMarine.getContext("2d");

    if (chart) {
      chart.destroy();
    }

    const labels = marineData.hourly.time.map((val)=>{
            return val.replace("T", " ");
        });
    const dataset1 = marineData.hourly.wave_height;
    const dataset2 = marineData.hourly.ocean_current_velocity;
    const label1 = "Wave height [m]";
    const label2 = "Ocean current velocity [km/h]";

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

<div id="mf_content">
    <div class="index_content bg-white bg-opacity-80 p-10 rounded-2xl shadow-lg min-w-2xl max-w-full text-center space-y-6">
      <div class="w-full flex flex-col items-center gap-4">
        <div class="flex flex-col sm:flex-row gap-2 w-full max-w-2xl">
          <input
            type="text"
            bind:value={location}
            placeholder="Enter location"
            class="p-3 w-full rounded border border-gray-400 text-gray-400 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
          />
        </div>

        <button
          on:click={fetchData}
          class="w-full p-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition"
        >
          Fetch marine forecast
        </button>

        {#if marineData}
          <div class="w-full max-w-4xl bg-white p-5">
            <canvas id="marineChart" height="400" width="800"></canvas>
          </div>

          <div class="mt-6 w-full overflow-auto max-h-[400px]">
            <table class="min-w-full text-gray-400 rounded-lg w-full">
              <thead class="bg-blue-400 text-white">
                <tr>
                  <th class="py-2 px-4">Date</th>
                  <th class="py-2 px-4">Wave height [m]</th>
                  <th class="py-2 px-4">Ocean current velocity [km/h]</th>
                </tr>
              </thead>
              <tbody>
                {#each marineData.hourly.time as time, indx}
                  <tr class="border-t text-gray-400">
                    <td class="py-2 px-4 text-center">{time.replace("T", " ")}</td>
                    <td class="py-2 px-4 text-center"
                      >{marineData.hourly.wave_height[indx]}</td
                    >
                    <td class="py-2 px-4 text-center"
                      >{marineData.hourly.ocean_current_velocity[indx]}</td
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
    #mf_content {
        width: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 3%;
        height: 100%;
    }
</style>

