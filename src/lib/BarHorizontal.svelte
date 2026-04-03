<script>
    import * as d3 from 'd3';

    let width = 600;
    let height = 300;
    export let data = [];
    export let title = "";

    let margin = { top: 20, right: 0, bottom: 10, left: 20 };
    let innerWidth  = width  - margin.left - margin.right;
    let innerHeight = height - margin.top  - margin.bottom;

    let xAxis, yAxis;

    // swap xScale and yScale defs
    $: yScale = d3.scaleBand()
        .domain(data.map(d => d.label))
        .range([0, innerHeight]) //changed innerWidth to innerHeight
        .padding(0.2);

    $: xScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.value) || 1])
        .range([0, innerWidth]); // switched 0 and inner 

    $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
        .domain(data.map(d => d.label));

    // swapped xAxis and yAxis d3.selects, swap axisLeft and axisBottom
    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(
            d3.axisBottom(xScale)
            .ticks(Math.min(d3.max(data, d => d.value), 10))
        );
        d3.select(yAxis).call(d3.axisLeft(yScale));
    }

    
    $: maxBar = d3.greatest(data, d => d.value);
</script>


<!-- add in html logic after fixing axes on regular bar -->

<div class="container">
    <svg viewBox="0 0 {width} {height}">
        <text
            x={margin.left + innerWidth / 2}
            y={margin.top / 2}
            text-anchor="middle"
            class="chart-title">
            {title}
        </text>
        <!-- x-axis label -->
        <text
            x={innerWidth / 2 + 40}
            y={innerHeight + margin.bottom + 60}
            text-anchor="middle"
            class="axis-label">
            Lines of Code
        </text>

        <!-- y-axis label -->
        <text
            x={-(innerHeight / 2) - 30}
            y={-margin.left - 30}
            text-anchor="middle"
            transform="rotate(-90)"
            class="axis-label">
            Programming Language
        </text>


        <g transform="translate({margin.left}, {margin.top + innerHeight})"
            bind:this={xAxis} />
        <g transform="translate({margin.left}, {margin.top})"
            bind:this={yAxis} />
        <g transform="translate({margin.left}, {margin.top})">
            {#each data as d}
                <rect
                    x={0}
                    y={yScale(d.label)}
                    width={xScale(d.value)}
                    height={yScale.bandwidth()}
                    fill={colorScale(d.label)}
                />
            {/each}

            {#if maxBar}
                <!-- highlight outline around the tallest bar -->
                <rect
                    x={0}
                    y={yScale(maxBar.label)}
                    width={xScale(maxBar.value)}
                    height={yScale.bandwidth()}
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                />
                <!-- leader line removed for lab 7 
                    <line
                        x1 = {xScale(maxBar.value) / 2 + 80}
                        y1 = {yScale(maxBar.label) + yScale.bandwidth()}
                        x2 = {xScale(maxBar.value) / 2 + 80}
                        y2 = {yScale(maxBar.label) + yScale.bandwidth() + 15}
                        
                        stroke="currentColor"
                        stroke-width="1"
                    />
                -->
                <!-- annotation text at end of leader line -->
                <text
                    x = {xScale(maxBar.value) + 5}
                    y = {yScale(maxBar.label) + yScale.bandwidth()/2 + 5}
                    text-anchor = "start"
                    class="annotation">
                    Most lines of code
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
</div>


<style>
    svg {
        max-width: 100%;
        width: 100%;
        height: 200px;
        overflow: visible;
        margin-bottom: 2em;
    }

    .container {
        display: flex;
        align-items: center;
        margin-top: 0em;
        overflow: visible;
    }

    .legend {
        flex: 1;
        font-size: 80%;
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
        font-size: 1.8em;
        font-weight: bold;
        fill: currentColor;
    }

    .axis-label {
        font-size: 1.2em;
        fill: currentColor;
    }

    .annotation {
        fill: currentColor;
        font-style: italic;
        margin-right: 1em;
    }

    g {
        font-size: 110%;
    }
</style>