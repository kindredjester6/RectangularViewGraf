<h1>Acerca de Nosotros</h1>
<p>¡Bienvenido a nuestra página de información!</p>

<script>
    import GrafFlare from "./grafFlare.svelte";
    import GrafVue from "./grafVue.svelte";
    import GrafDist from "./grafDistCr.svelte"
</script>

<style>
    div{
        display: flex;
        align-items: flex-start;
        flex-wrap: wrap;
        flex-direction: column;
        box-shadow: 0rem 0rem 0.5rem rgba(0, 0, 0, 0.5rem);
    }
</style>

<div >
<GrafFlare />

<GrafVue />

<GrafDist />
</div>