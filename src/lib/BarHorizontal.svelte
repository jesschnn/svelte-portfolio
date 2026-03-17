<script>
    import * as d3 from 'd3';

    let width = 400;
    let height = 300;
    export let data = [];

    let margin = { top: 40, right: 85, bottom: 80, left: 60 };
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
        d3.select(yAxis).call(d3.axisLeft(yScale));
        d3.select(xAxis).call(
            d3.axisBottom(xScale)
                .tickFormat(d => Number.isInteger(d) ? d : "")
                .ticks(5)
        );
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
            Lines of Code by Language
        </text>
        <!-- x-axis label -->
        <text
            x={innerWidth / 2 + 50}
            y={innerHeight + margin.bottom}
            text-anchor="middle"
            class="axis-label">
            Lines of Code
        </text>

        <!-- y-axis label -->
        <text
            x={-(innerHeight / 2) - 40}
            y={-margin.left + 60}
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
                <!-- leader line -->
                <line
                    x1 = {xScale(maxBar.value) / 2}
                    y1 = {yScale(maxBar.label) + yScale.bandwidth()}
                    x2 = {xScale(maxBar.value) / 2}
                    y2 = {yScale(maxBar.label) + yScale.bandwidth() + 15}
                    
                    stroke="currentColor"
                    stroke-width="1"
                />
                <!-- annotation text at end of leader line -->
                <text
                    x = {xScale(maxBar.value) / 2 - 40}
                    y = {yScale(maxBar.label) + yScale.bandwidth() + 25}
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
        max-width: 80%;
        height: auto;
        overflow: visible;
        margin: 0;
    }


    .container {
        display: flex;
        align-items: center;
        margin-top: 1em;
    }

    .legend {
        flex: 1;
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
</style>