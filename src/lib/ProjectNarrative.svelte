<script>
    import Scrolly from "svelte-scrolly";
    let scrollyProgress = 0;
    import projects from "./projects.json";
    let sorted_projects = projects.sort((a, b) => a.year - b.year);
    let progressPerProject = 100 / sorted_projects.length;
    $: activeProjectIdx = Math.min(sorted_projects.length-1, Math.floor(scrollyProgress / progressPerProject));

    import { base } from "$app/paths";
</script>

<div class="scrolly-wrapper">
    <Scrolly bind:progress={ scrollyProgress }>
            <!-- Story here -->
            {#each sorted_projects as p}
                <section class="step">
                    <div class="step-content">
                        <h3> {p.title} </h3>
                        <p> {p.story} </p>
                    </div>
                </section>
            {/each}
            
            <svelte:fragment slot="viz">
                <!-- Visualizations here (these will stay sticky) -->
                <div class="project-detail">
                    <h3> {sorted_projects[activeProjectIdx].year} </h3>
                    <img src={base[-1] == "/" ?
                            base + sorted_projects[activeProjectIdx].image :
                            base + "/" + sorted_projects[activeProjectIdx].image} 
                        alt="Image of {sorted_projects[activeProjectIdx].title}"> 
                </div>
                
            </svelte:fragment>
    </Scrolly>
</div>

<style>
    .scrolly-wrapper {
        width: min(1000ch, 70vw);
        position: relative;
        left: 60%;
        transform: translateX(-50%);
    }

    .step {
        min-height: 50vh;
        padding: 2rem;
    }

    .step-content {
        border-left: 0.5em solid color-mix(in oklch, var(--color-accent), canvas 85%);
        padding: 1.5rem 2rem;
    }

    .project-detail {
        padding: 2rem;
        width: 100%;
    }
</style>