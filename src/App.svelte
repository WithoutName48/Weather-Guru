<script>
  import Router from "svelte-spa-router";
  import Index from "./pages/Index.svelte";
  import Forecast from "./pages/Forecast.svelte";
  import HistoricalWeather from "./pages/HistoricalWeather.svelte";
  import AirQuality from "./pages/AirQuality.svelte";
  import MarineForecast from "./pages/MarineForecast.svelte";

  const routes = {
    "/": Index,
    "/forecast": Forecast,
    "/history": HistoricalWeather,
    "/air-quality": AirQuality,
    "/marine": MarineForecast,
  };
</script>

<!-- <main
  id="mane"
  class="flex flex-col flex-1 items-center bg-gray-600 justify-around"
> -->
<Router {routes} />
<!-- </main> -->

<style lang="postcss">
  @reference "tailwindcss";
</style>