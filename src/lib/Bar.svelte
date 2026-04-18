<script>
    import * as d3 from 'd3';

    let width = 400;
    let height = 300;
    /* let data = [
        { label: "A", value: 10 },
        { label: "B", value: 20 },
        { label: "C", value: 15 },
        { label: "D", value: 8 },
        { label: "E", value: 25 }
    ];*/
    export let data = [];

    let margin = { top: 40, right: 85, bottom: 80, left: 50 };
    let innerWidth  = width  - margin.left - margin.right;
    let innerHeight = height - margin.top  - margin.bottom;

    let xAxis, yAxis;

    $: xScale = d3.scaleBand()
        .domain(data.map(d => d.label))
        .range([0, innerWidth])
        .padding(0.2);

    $: yScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.value) || 1])
        .range([innerHeight, 0]);

    $: colorScale = d3.scaleOrdinal()
        .domain(data.map(d => d.label))
        .range(d3.quantize(d3.interpolateBlues, data.length));


    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(
            d3.axisLeft(yScale)
                .tickFormat(d => Number.isInteger(d) ? d : "")
                .tickValues(d3.range(0, d3.max(data, d => d.value) + 1))
        );
    }
    
    $: maxBar = d3.greatest(data, d => d.value);

    /* lab 10 */
    let selectedIndex = -1;
    let liveText = "";

    function toggleBar(index, event) {
        if (!event.key || event.key === "Enter") {
            selectedIndex = index;
            const d = data[index];
            liveText = `${d.label}: ${d.value} projects selected.`;
        }
    }

    $: description = `A bar chart showing project counts by year. ${data.map(d => `${d.label}: ${d.value} projects`).join(', ')}.`;
    let showChart = true;

    function toggleView() {
        showChart = !showChart;
        liveText = showChart ? "Bar chart view shown." : "Table view shown.";
    }


</script>


<button
  on:click={toggleView}
  aria-pressed={!showChart}
  aria-label="Toggle between bar chart and table view"
  class="toggle-button">
    {showChart ? 'Show Table' : 'Show Chart'}
</button>
{#if showChart}
    <div class="container">
        <svg 
            viewBox="0 0 {width} {height}"
            role="img"
            aria-labelledby="bar-title bar-desc">
            <title id="bar-title">Projects by Year</title>
            <desc id="bar-desc">{description}</desc>
            >
            <!-- your code for the bars --> 
            <text
                x={margin.left + innerWidth / 2}
                y={margin.top / 2}
                text-anchor="middle"
                class="chart-title">
                Projects per Year
            </text>
            <!-- x-axis label -->
            <text
                x={innerWidth / 2 + 50}
                y={innerHeight + margin.bottom }
                text-anchor="middle"
                class="axis-label">
                Year
            </text>

            <!-- y-axis label -->
            <text
                x={-(innerHeight / 2) - 30}
                y={-margin.left + 60}
                text-anchor="middle"
                transform="rotate(-90)"
                class="axis-label">
                Number of Projects
            </text>

            <g transform="translate({margin.left}, {margin.top + innerHeight})"
                bind:this={xAxis} />
            <g transform="translate({margin.left}, {margin.top})"
                bind:this={yAxis} />
            <g transform="translate({margin.left}, {margin.top})">
                {#each data as d, index}
                    <rect
                        x={xScale(d.label)}
                        y={yScale(d.value)}
                        width={xScale.bandwidth()}
                        height={innerHeight - yScale(d.value)}
                        fill={colorScale(d.label)}
                        opacity={selectedIndex === -1 || selectedIndex === index ? 1 : 0.45}
                        on:click={e => toggleBar(index, e)}
                        on:keyup={e => toggleBar(index, e)}
                        tabindex="0"
                        role="button"
                        aria-label="bar"
                        stroke="black"
                    />
                {/each}

                {#if maxBar}
                    <!-- highlight outline around the tallest bar -->
                    <rect
                        x={xScale(maxBar.label)}
                        y={yScale(maxBar.value)}
                        width={xScale.bandwidth()}
                        height={innerHeight - yScale(maxBar.value)}
                        fill="none"
                        stroke="currentColor"
                        stroke-width="2"
                    />
                    <!-- leader line -->
                    <line
                        x1={xScale(maxBar.label) + xScale.bandwidth()}
                        y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 4 - 20}
                        x2={xScale(maxBar.label) + xScale.bandwidth() + 30}
                        y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 4 - 20}
                        stroke="currentColor"
                        stroke-width="1"
                    />
                    <!-- annotation text at end of leader line -->
                    <text
                        x={xScale(maxBar.label) + xScale.bandwidth() + 35}
                        y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 4 - 20}
                        dominant-baseline="middle"
                        class="annotation">
                        Year with most projects
                    </text>
                {/if}
            </g>
        </svg>
        <ul class="legend">
            {#each data as d}
                <li style="--color: {colorScale(d.label)}">
                    <span class="swatch"></span>
                    {d.label} <em>({d.value})</em>
                </li>
            {/each}
        </ul>
        <p aria-live="polite" class="sr-only">{liveText}</p>
    </div>
{:else}
    <table aria-label="Table showing project counts by year" class="data-table">
        <caption>Projects by Year</caption>
        <thead>
            <tr>
            <th id="year-header" scope="col">Year</th>
            <th id="projects-header" scope="col">Projects</th>
            </tr>
        </thead>
        <tbody>
            {#each data as d, i}
            <tr>
                <th id="row-{i}" scope="row">{d.label}</th>
                <td aria-labelledby="row-{i} projects-header">{d.value}</td>
            </tr>
            {/each}
        </tbody>
    </table>
{/if}



<style>
    svg {
        max-width: 80%;
        height: auto;
        overflow: visible;
        margin: 0;
        flex: 0 0 auto;
    }

    .container {
        display: flex;
        flex-wrap: wrap;
        align-items: center;
        width: 100%;
        gap: 0em;
    }


    .legend {
        flex: 1;
        margin-left: 0em;
    }

    .swatch {
        width: 10px;
        height: 10px;
        background-color: var(--color);
    }

    li {
        display: flex;
        align-items: center;
        gap: 0.5em;
    }

    .chart-title {
        font-size: 1em;
        font-weight: bold;
        fill: currentColor;
    }

    .axis-label {
        font-size: 0.8em;
        fill: currentColor;
    }

    .annotation {
        font-size: 0.7em;
        fill: currentColor;
        font-style: italic;
    }

    rect {
        transition: 300ms;
        outline: none;
        /* stroke: currentColor; */
        /* stroke-width: 1; */
    }

    svg:hover rect:not(:hover), .container:focus-within rect:not(:focus-visible) { opacity: 50%; }
    rect:focus-visible {
        stroke: currentColor;
        stroke-width: 2px;
        stroke-dasharray: 4;
    }

    .sr-only {
        position: absolute;
        left: -9999px;
        width: 1px;
        height: 1px;
        overflow: hidden;
    }  

    .data-table {
        margin-top: 1rem;
        margin-bottom: 1rem;
        border-collapse: collapse;
        width: 100%;
        max-width: 30em;
    }

    .data-table caption {
        font-weight: bold;
        margin-bottom: 0.5em;
        text-align: left;
    }

    .data-table th,
    .data-table td {
        border: 1px solid #ccc;
        padding: 0.5em;
        text-align: left;
        color: #000;
        background-color: white;
    }

    .data-table th {
        background-color: #f0f0f0;
    }

</style>