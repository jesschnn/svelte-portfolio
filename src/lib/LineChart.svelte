<script>
    import * as d3 from 'd3';
    export let data = [];

    let width = 1000, height = 300;
    // copied margin and usableArea from meta page
    let margin = { top: 20, right: 20, bottom: 0, left: 0 };
    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    };

    let xAxis, yAxis;

    $: line = d3.line()
        .x(d => xScale(d.date))
        .y(d => yScale(d.count))
        .curve(d3.curveBumpX);

    $: xScale = d3.scaleTime()
        .domain(d3.extent(data, d => d.date))
        .range([usableArea.left, usableArea.right]);

    $: yScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.count)])
        .range([usableArea.bottom, usableArea.top])
        .nice();

    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(d3.axisBottom(xScale)
            .ticks(d3.timeDay.every(7))
            .tickFormat(d3.timeFormat("%b %d"))
        );

        d3.select(yAxis).call(d3.axisLeft(yScale)
            .ticks(5)
            .tickSizeOuter(0)
        );
    }

    let hoveredDay = null; // e.g. "Monday"
    $: dayRegions = (() => {
        // if there's no data, there are no regions!
        if (data.length === 0) return [];
        return data.map((d, i, arr) => {
            // get the previous date, if it exists
            const prev = arr[i - 1];
            // get the next date, if it exists
            const next = arr[i + 1];
            // define the left side of the region, which might be the edge of the axis if this is the first date
            const left = prev ? new Date((d.date.getTime() + prev.date.getTime()) / 2) : d.date;
            // define the right side of the region, which might be the edge of the axis if this is the last date
            const right = next ? new Date((d.date.getTime() + next.date.getTime()) / 2) : d.date;
            
            return {
                date: d.date,
                weekday: d.date.toLocaleString("en", { weekday: "long" }), // e.g. "Monday"
                x: xScale(left),
                width: xScale(right) - xScale(left),
            };
        });

    })();

</script>


<div class="line-chart">
    <h3>
        Lines Edited
        {#if hoveredDay === null}
            by Day
        {:else}
            on {hoveredDay}
        {/if}
    </h3>
    <svg viewBox="0 0 {width} {height}" on:mouseleave={() => hoveredDay = null}>
        <!-- line chart will go here -->
        <path
            d={line(data)}
            fill="none"
            stroke="steelblue"
            stroke-width="2"
        />
        <!-- dots at each data point -->
        {#each data as d}
            {@const isHighlighted = d.date.toLocaleString("en", { weekday: "long" }) === hoveredDay}
            <circle
                cx={xScale(d.date)}
                cy={yScale(d.count)}
                r="3"
                fill="steelblue"
            />

            {#if isHighlighted}
                <text
                    x={xScale(d.date)}
                    y={usableArea.top + 15}
                    text-anchor="middle"
                    font-size="12"
                    fill="var(--color-accent)"
                >
                    {Math.round(d.count)}
                </text>

                <circle
                    cx={xScale(d.date)}
                    cy={yScale(d.count)}
                    r="6"
                    fill="var(--color-accent)"
                />
            {/if}
        {/each}
        
        <!-- x-axis label -->
        <text
            x={usableArea.left + (usableArea.right - usableArea.left) / 2}
            y={height + 35}
            text-anchor="middle"
            class="axis-label">
            Date
        </text>

        <!-- y-axis label -->
        <text
            x={-(usableArea.top + (usableArea.bottom - usableArea.top) / 2) - 30}
            y={-50}
            text-anchor="middle"
            transform="rotate(-90)"
            class="axis-label">
            Number of Lines Edited
        </text>

        <!-- axes lines -->
        <g transform="translate({margin.left}, {usableArea.bottom})"
            bind:this={xAxis} />
        <g transform="translate({margin.left}, 0)"
            bind:this={yAxis} />

        <!-- invisible hover regions -->
        {#each dayRegions as region}
            <rect
                x={region.x}
                y={usableArea.top}
                width={region.width}
                height={usableArea.bottom - usableArea.top}
                fill="transparent"
                on:mouseenter={() => hoveredDay = region.weekday}
            />
            {#if region.weekday === hoveredDay}
                <rect
                    x={region.x}
                    y={usableArea.top}
                    width={region.width}
                    height={usableArea.bottom - usableArea.top}
                    fill="var(--color-accent)"
                    opacity="0.2"
                />
            {/if}
        {/each}
    </svg>
</div>


<style>
	svg {
        max-width: 100%;
        height: auto;
        overflow: visible;
        margin-bottom: 2em;
        flex: 0 0 auto;
    }


    h3 {
        text-align: center;
    }

    .axis-label {
        font-size: em;
        fill: currentColor;
    }

</style>