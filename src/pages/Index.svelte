<script>
    import Template from "../components/Template.svelte";
</script>

<div id="index">
    <Template></Template>
    <div id="index_content">AAA</div>
</div>

<style>
    #index {
        width: 100vw;
        height: 100vh;
        background-image: url('https://ukhsa.blog.gov.uk/wp-content/uploads/sites/33/2023/06/rainbow-4047523_1920.jpg');
        background-size: cover;
        background-position: center center;

        display: flex;
        flex-direction: row;
    }

    #index_content {
        width: 100%;

        display: flex;
        justify-content: center;
        align-items: center;
    }
</style>