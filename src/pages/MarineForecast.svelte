<script>
    import Template from "../components/Template.svelte";
    import MFContent from "../components/MFContent.svelte";
</script>

<div id="index" class="w-full min-h-screen flex flex-col lg:flex-row bg-cover bg-center">
    <div class="w-full lg:w-[30%] lg:fixed top-0 left-0">
    <Template />
    </div>

    <div class="w-full lg:w-[70%] ml-auto">
    <MFContent />
    </div>
</div>

<style>
  #index {
    background-image: url("https://preview.redd.it/nduksi52zrs41.jpg?auto=webp&s=0760039b9b3ebdb699ef6569ee313b787f27375c");
  }
</style>
