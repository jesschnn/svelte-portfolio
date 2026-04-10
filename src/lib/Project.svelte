<script>
    export let data = {};
    import { base } from "$app/paths";
</script>

<article>
    <h2>{data.title}</h2>
    <img src={base[-1] == "/" ? (base + data.image) : (base + "/" + data.image)} alt="">
    <p>{data.year}: {data.description}</p>
    {#if data.url}
        <a class="reg-link" href={data.url}>Visit website</a>
    {/if}
</article>


<style>
    img {
    max-width: 100%;
    }

    article {
    display: grid;
    grid-template-rows: subgrid;
    grid-row: span 4;
    gap: 0.3em;
    }

    h2 {
    margin: 0;
    font-size: 1.5em;
    }

    h1, h2, h3, h4, h5, h6 {
    line-height: 1.1;
    text-wrap: balance;
    }

    .reg-link {
        text-decoration: underline;
        text-align: left;
        padding: 0;
        color: blue;
    }

    a:hover {
        border-bottom: none;
        background-color: transparent;
    }
</style>
