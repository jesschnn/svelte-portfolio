<script>
    import { base } from "$app/paths";
    import { page } from "$app/stores";

    let pages = [
        {url: "/", title: "About"},
        {url: "/projects", title: "Projects"},
        {url: "/resume", title: "Resume"},
        {url: "/contact", title: "Contact"},
        {url: "https://github.com/jesschnn", title: "Github"}
        ];

    let colorScheme = "light dark"; // default colorScheme value
    let root = globalThis.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);
    
    let localStorage = globalThis.localStorage ?? {};
    if (localStorage.colorScheme) { // if localStorage has a colorScheme property
        colorScheme = localStorage.colorScheme; // override the default colorScheme
    }
    
    $: localStorage.colorScheme = colorScheme;
</script>

<label class="color-scheme-switch">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>
<br>
<nav>
    {#each pages as p}
        <a href={base + p.url}
            class:current={p.url === "/" // is this link the home page?
            ? $page.url.pathname === (base + "/") // if yes - set current = true if the path name matches. Else, set current = true if the path name starts correctly
            : $page.url.pathname.startsWith(base + p.url)}
            target={p.url.startsWith("https") ? "_blank": null}
        >
            {p.title}
        </a>
    {/each}
</nav>

<style>
    :root {
        color-scheme: light dark;
    }

    nav {
        --border-color: oklch(50% 10% 200 / 40%);
            /* ... other styles and nested rules ... */
        border-bottom: 2px solid var(--border-color);
    }
    
    .current {
        border-bottom: 4px solid var(--border-color); 
        background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
    }

    a:hover {
        background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
    }

    label.color-scheme-switch {
        position: absolute;
        top: 0.5rem;
        right: 1rem;
    }

    .color-scheme-switch {
        display: inline-flex;
        gap: 4px;
        font-size: 80%;
        font: inherit;
    }    

</style>

<slot />