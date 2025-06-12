<script>
    // @ts-nocheck
    import Template from "../components/Template.svelte";

    import { tick } from "svelte";
    import Chart from "chart.js/auto";

    // european_aqi_pm2_5
    // european_aqi_pm10
    // european_aqi_nitrogen_dioxide
    // european_aqi_ozone
    // european_aqi_sulphur_dioxide

    const EAQI = [
        [10, 20, 25, 50, 75, 800],
        [20, 40, 50, 100, 150, 1200],
        [40, 90, 120, 230, 340, 1000],
        [50, 100, 130, 240, 380, 800],
        [100, 200, 350, 500, 750, 1250],
    ];

    const EAQI_mark = [
        "very good",
        "good",
        "medium",
        "poor",
        "very poor",
        "extremely poor",
    ];
    const EAQI_color = [
        "green",
        "yellowgreen",
        "yellow",
        "orange",
        "orangered",
        "red",
    ];

    let location = "";
    let latitude = null;
    let longitude = null;
    let airQualityData = null;
    let chart;
    let tableData = null;

    // Function to get the coordinates of the location
    async function getCoordinates() {
        const res = await fetch(
            `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(location)}&count=1`,
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

        const url = `https://air-quality-api.open-meteo.com/v1/air-quality?latitude=${latitude}&longitude=${longitude}&hourly=european_aqi,european_aqi_pm2_5,european_aqi_pm10,european_aqi_nitrogen_dioxide,european_aqi_ozone,european_aqi_sulphur_dioxide&forecast_days=7`;

        const res = await fetch(url);
        airQualityData = await res.json();

        await tick(); // wait for DOM update

        // After fetching, draw chart
        drawChart();
    }

    // Function to draw the forecast chart
    function drawChart() {
        if (!airQualityData) return;

        const canvasAirQuality = document.getElementById("airQualityChart");
        const ctx = canvasAirQuality.getContext("2d");

        if (chart) {
            chart.destroy();
        }

        const dataToMap = Object.entries(airQualityData.hourly).filter((val) =>
            val[0].includes("european"),
        );

        console.log(dataToMap.filter((val) => val[0] !== "european_aqi"));

        const tableData2 = dataToMap
            .filter((val) => val[0] !== "european_aqi")
            .map(([key, arrData], indx) => {
                arrData = arrData.filter((val) => val !== null);
                const newArr = arrData.map((val) => {
                    let valLabel = null;
                    console.log(indx);
                    EAQI[indx].forEach((val_EAQI, j) => {
                        if (valLabel === null && val < val_EAQI) {
                            valLabel = j;
                        }
                    });
                    return valLabel;
                });
                return newArr;
            });

        console.log(tableData2);

        const tableData1 = [];
        for (let i = 0; i < tableData2.length; i++) {
            for (let j = 0; j < tableData2[i].length; j++) {
                if (tableData1.length <= j) {
                    tableData1.push(tableData2[i][j]);
                } else if (tableData1[j] < tableData2[i][j]) {
                    tableData1[j] = tableData2[i][j];
                }
            }
        }

        console.log(tableData1);

        tableData = [tableData1, ...tableData2];

        console.log(tableData);

        const datasets = dataToMap.map(([key, val], indx) => {
            const newArr = val.filter((val) => {
                return val !== null;
            });
            return {
                [key]: newArr,
            };
        });

        const labels = airQualityData.hourly.time;
        const labels_datasets = [
            "European AQI",
            "European AQI PM2.5",
            "European AQI PM10",
            "European AQI NO2",
            "European AQI O3",
            "European AQI SO2",
        ];

        chart = new Chart(ctx, {
            type: "line",
            data: {
                labels,
                datasets: [
                    {
                        label: labels_datasets[0],
                        data: datasets[0].european_aqi,
                        borderColor: "blue",
                        backgroundColor: "lightblue",
                        fill: false,
                    },
                    {
                        label: labels_datasets[1],
                        data: datasets[1].european_aqi_pm2_5,
                        borderColor: "red",
                        backgroundColor: "pink",
                        fill: false,
                    },
                    {
                        label: labels_datasets[2],
                        data: datasets[2].european_aqi_pm10,
                        borderColor: "green",
                        backgroundColor: "lightgreen",
                        fill: false,
                    },
                    {
                        label: labels_datasets[3],
                        data: datasets[3].european_aqi_nitrogen_dioxide,
                        borderColor: "gray",
                        backgroundColor: "lightgray",
                        fill: false,
                    },
                    {
                        label: labels_datasets[4],
                        data: datasets[4].european_aqi_ozone,
                        borderColor: "orange",
                        backgroundColor: "orangered",
                        fill: false,
                    },
                    {
                        label: labels_datasets[5],
                        data: datasets[5].european_aqi_sulphur_dioxide,
                        borderColor: "purple",
                        backgroundColor: "violet",
                        fill: false,
                    },
                ],
            },
            options: {
                responsive: true,
                plugins: {
                    title: {
                        display: true,
                        text: "European AQI Data Over Time",
                    },
                    legend: {
                        position: "bottom",
                    },
                },
                scales: {
                    x: {
                        display: true,
                        title: {
                            display: true,
                            text: "Time Index",
                        },
                    },
                    y: {
                        display: true,
                        title: {
                            display: true,
                            text: "AQI Value",
                        },
                    },
                },
            },
        });
    }
</script>

<div id="aq_content">
    <div class="bg-white bg-opacity-80 p-10 rounded-2xl shadow-lg max-w-2xl text-center space-y-6">
        <div class="w-full flex flex-col items-center gap-4">
            <div class="flex flex-col sm:flex-row gap-2 w-full max-w-2xl">
                <input
                    type="text"
                    bind:value={location}
                    placeholder="Enter location"
                    class="p-3 w-full rounded border border-gray-300 text-gray-300 focus:text-blue-400 focus:outline-none focus:ring-2 focus:ring-blue-400"
                />
            </div>

            <button
                on:click={fetchData}
                class="w-full p-3 !bg-blue-500 text-white rounded-lg hover:bg-blue-600 transitio"
            >
                Fetch air quality
            </button>

            {#if airQualityData}
                <div class="w-full max-w-4xl">
                    <canvas id="airQualityChart" height="400" width="800"
                    ></canvas>
                </div>

                <div class="mt-6 w-full overflow-auto max-h-[500px]">
                    <table class="min-w-full bg-white rounded-lg w-full">
                        <thead class="bg-gray-200">
                            <tr>
                                <th class="py-2 px-4 bg-blue-400 text-white"
                                    >Date</th
                                >
                                <th class="py-2 px-4 bg-blue-400 text-white"
                                    >European AQI</th
                                >
                                <th class="py-2 px-4 bg-blue-400 text-white"
                                    >European AQI PM<sub>2.5</sub></th
                                >
                                <th class="py-2 px-4 bg-blue-400 text-white"
                                    >European AQI PM<sub>10</sub></th
                                >
                                <th class="py-2 px-4 bg-blue-400 text-white"
                                    >European AQI NO<sub>2</sub></th
                                >
                                <th class="py-2 px-4 bg-blue-400 text-white"
                                    >European AQI O<sub>3</sub></th
                                >
                                <th class="py-2 px-4 bg-blue-400 text-white"
                                    >European AQI SO<sub>2</sub></th
                                >
                            </tr>
                        </thead>
                        <tbody>
                            {#each airQualityData.hourly.time as time, i}
                                <tr class="border-t">
                                    <td
                                        class="py-2 px-4 text-cent bg-gray-600 text-white"
                                        >{time}</td
                                    >
                                    {#each tableData as arr}
                                        {#if i < arr.length}
                                            <td
                                                class="py-2 px-4 text-center bg-gray-800"
                                                style="color: {EAQI_color[
                                                    arr[i]
                                                ]};"
                                            >
                                                {EAQI_mark[arr[i]]}
                                            </td>
                                        {:else}
                                            <td
                                                class="py-2 px-4 text-center bg-gray-800 text-white"
                                            >
                                                no data
                                            </td>
                                        {/if}
                                    {/each}
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
    #aq_content {
        width: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 3%;
        height: 100%;
    }
</style>