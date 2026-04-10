<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";

  // lab 4
  import { onMount } from "svelte";

  let githubData = null; // This will eventually hold our Github stats
  let loading = true; // This will be true *until* the fetch's promise resolves to a value
  let error = null; // If the API call resulted in an error, it will go into this variable

  
  onMount(async () => {
    try {
        console.log("Page has been mounted!")
        let response = await fetch("https://api.github.com/users/jesschnn");
        /*
        let response = await {
            ok: true,
            json: async () => ({
                followers: 100,
                following: 100,
                public_repos: 100,
                public_gists: 100,
            }),
        }; */
        console.log(response);
        githubData = await response.json();
        console.log(githubData);
    } catch (err) {
        error = err;
    }
    loading = false;
  });

</script>



<h1>Jessica Chen</h1>


<div class="intro-section">
    <div class="intro">
        <p>Hi! I'm Jessica, and I like watching shows and taking naps.</p>
        <img src="images/jessica_jumping.JPG" alt="Picture of me jumping">
    </div>

    <div class="reading">
        <h2>What I'm Reading</h2>
        <div class="readings">
            {#each reading as b}
                <ReadingItem data={b} />
            {/each}
        </div>
    </div>
</div>
<hr>

<div class="github-stats">
    {#if loading}
        <p>Loading...</p>
    {:else if error}
        <p>Something went wrong: {error.message}</p>
    {:else}
        <section>
            <h2>My GitHub Stats</h2>
            <dl>
                <dt>FOLLOWERS</dt>
                <dd>{githubData.followers}</dd>
                <dt>FOLLOWING</dt>
                <dd>{githubData.following}</dd>
                <dt>PUBLIC REPOSITORIES</dt>
                <dd>{githubData.public_repos}</dd>
            </dl>
        </section>
    {/if}
</div>

<div class="project-highlights">
    <br>
    <h2>Latest Projects</h2>
    <br>

    <div class="projects">
        {#each projects.slice(7, 10) as p}
            <Project data={p} />
        {/each}
    </div>
</div>


<style>
    .intro-section {
        display: flex;
        gap: 2em;
    }

    .intro {
        flex: 1.25;
    }

    .reading {
        flex: 0.75;
        background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
        padding: 1em;
    }

    dl {
        display: grid;
        grid-template-columns: repeat(3,1fr);
        gap: 10px;
    }

    dt {
        grid-row: 1;
        color: grey;
        text-align: left;
    }

    dd {
        grid-row: 2;
        font-size: 150%;
        margin-left: 0;
    }
</style>


