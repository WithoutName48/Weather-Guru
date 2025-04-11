<script>
  // @ts-ignore
  import Router from "svelte-spa-router";
  import Page from "./screens/Template.svelte";
  import Page2 from "./screens/Template.svelte";

  const routes = {
    "/": Page,
    "/page2": Page2,
  };
</script>

<!-- <main
  id="mane"
  class="flex flex-col flex-1 items-center bg-gray-600 justify-around"
> -->
  <Router {routes} />
<!-- </main> -->
