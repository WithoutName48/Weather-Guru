<script>
  import Router from "svelte-spa-router";
  import Index from "./pages/Index.svelte";
  import Forecast from "./pages/Forecast.svelte";

  const routes = {
    "/": Index,
    "/forecast": Forecast,
  };
</script>

<!-- <main
  id="mane"
  class="flex flex-col flex-1 items-center bg-gray-600 justify-around"
> -->
  <Router {routes} />
<!-- </main> -->
