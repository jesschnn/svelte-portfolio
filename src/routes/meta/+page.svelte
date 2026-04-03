<svelte:head>
  <title>Meta</title>
</svelte:head>

<script>
    import { base } from '$app/paths';
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import BarHorizontal from '$lib/BarHorizontal.svelte';
    import LineChart from '$lib/LineChart.svelte';

    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';

    let locData = [];
    let commits = [];
    let width = 1000, height = 400;
    let xAxis, yAxis;
    let yAxisGridlines;
    let margin = { top: 20, right: 20, bottom: 0, left: 0 };
    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;

    onMount(async () => {
        locData = await d3.csv(`${base}/loc.csv`, row => ({
            ...row,
            line: Number(row.line),
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));

        commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
            let first = lines[0];
            let {author, date, time, timezone, datetime} = first;
            let ret = {
                id: commit,
                url: "https://github.com/vis-society/lab-7/commit/" + commit,
                author, date, time, timezone, datetime,
                hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                totalLines: lines.length,
                lines: lines
            };

            return ret;
        });

        commits = d3.sort(commits, d => -d.totalLines);

    });

    /* from lab 6 */
    /* $: locBarData = d3.rollups(locData, v => v.length, d => d.type)
        .map(([type, count]) => ({ label: String(type), value: count })); */

    /* new definitions for lab 7 */
    // 1.
    // $: selectedLines = (clickedCommits.length > 0 ? clickedCommits.flatMap(d => d.lines) : locData);

    //2.
    $: selectedCounts = d3.rollup(
        selectedLines,
        v => v.length,
        d => d.type
    );

    //3.
    $: allTypes = Array.from(new Set(locData.map(d => d.type)));

    //4.
    $: locBarData = allTypes.map(type =>  ({ label: String(type), value: selectedCounts.get(type) || 0 }));
    $: title = selectedCommits.length > 0 ? `Lines of Code: ${selectedCommits.length} Selected commits` : "Lines of Code: Website Breakdown";
    
    $: [minDate, maxDate] = d3.extent(commits.map(d => d.date));
    $: maxDatePlusOne = new Date(maxDate);
    $: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

    $: xScale = d3.scaleTime()
                .domain([minDate, maxDatePlusOne])
                .range([0, width])
                .nice();

    $: yScale = d3.scaleLinear()
                .domain([24, 0])
                .range([height, 0]);

    
    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale)
            .tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
    }

    $: {
        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
            .tickFormat("")
            .tickSize(-usableArea.width)
        );
    }

    $: rScale = d3.scaleSqrt([5,30], [5, 10]);

    let hoveredIndex = -1;
    $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

    let commitTooltip;
    let tooltipPosition = {x: 0, y: 0};
    let clickedCommits = [];

    async function dotInteraction (index, evt) {
        let hoveredDot = evt.target;
        if (evt.type === "mouseenter") {
            // dot hovered
            hoveredIndex = index;
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: "fixed", // because we use position: fixed
                middleware: [
                    offset(5), // spacing from tooltip to dot
                    autoPlacement() // see https://floating-ui.com/docs/autoplacement
                ],
            });        }
        else if (evt.type === "mouseleave") {
            // dot unhovered
            hoveredIndex = -1
        }
        else if (evt.type === "click") {
            let commit = commits[index]
            if (!clickedCommits.includes(commit)) {
                // Add the commit to the clickedCommits array
                clickedCommits = [...clickedCommits, commit];
            }
            else {
                    // Remove the commit from the array
                    clickedCommits = clickedCommits.filter(c => c !== commit);
            }
        }
    }

    // start of lab 8 
    let svg; 
    $: if (svg && usableArea) {
        d3.select(svg).call(d3.brush()
            .extent([[usableArea.left, usableArea.top], [usableArea.right, usableArea.bottom]])
            .on("start brush end", brushed)); 
        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }

    $: brushSelection = null;
    function brushed (evt) {
        brushSelection = evt.selection;
    }

    function isCommitBrushed (commit) {
        if (!brushSelection) {
            return false;
        }
        // TODO return true if commit is within brushSelection
        // and false if not
        let min = {x: brushSelection[0][0], y: brushSelection[0][1]};
        let max = {x: brushSelection[1][0], y: brushSelection[1][1]};
        let x = xScale(commit.date);
        let y = yScale(commit.hourFrac);
        return x >= min.x && x <= max.x && y >= min.y && y <= max.y;
    }
    $: brushedCommits = brushSelection ? commits.filter(isCommitBrushed) : [];
    $: selectedCommits = Array.from(new Set([...clickedCommits, ...brushedCommits]));
    $: selectedLines = (selectedCommits.length > 0 ? selectedCommits : commits).flatMap(d => d.lines);

    let linesByDate = [];
    $: {
        // 1. Get the count for each date in the data
        const rolled = d3.rollups(
            locData,
            v => v.length,
            d => d3.timeDay.floor(d.datetime)
        ).map(([date, count]) => ({ date, count }));

        // 2. Get an array of all days covered by the data
        const [minDate, maxDate] = d3.extent(rolled, d => d.date);
        const allDays = d3.timeDays(minDate, d3.timeDay.offset(maxDate, 1));

        // 3. Build linesByDate by filling all undefined dates with 0 counts
        linesByDate = allDays.map(date => ({
            date,
            count: rolled.find(d => d.date.getTime() === date.getTime())?.count ?? 0
        }));

        // console.log(linesByDate);
    }


</script>

<h1>Meta</h1>

<div> Meta page to visualize project data.</div>

<div class="charts">
    <div class="toolplot">
        <div class="scatterplot">
            <h3 class="scattertitle">Commits by time of day</h3>
            <svg viewBox="0 0 {width} {height}" bind:this={svg}>
                <!-- gridlines -->
                <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />

                <!-- the axes -->
                <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
                <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
                
                <!-- the plotted points-->
                <g class="dots">
                {#each commits as commit, index }
                    <circle
                        on:mouseenter={evt => dotInteraction(index, evt)}
                        on:mouseleave={evt => dotInteraction(index, evt)}
                        on:click={ evt => dotInteraction(index, evt) }
                        class:selected={ selectedCommits.includes(commit) }
                        
                        cx={ xScale(commit.date) }
                        cy={ yScale(commit.hourFrac) }
                        r={ rScale(commit.totalLines) }
                        fill="steelblue"
                    />
                {/each}
                </g>
            </svg>
        </div>
        <dl class="info tooltip" hidden={hoveredIndex === -1} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px" bind:this={commitTooltip}>
                <dt>COMMIT</dt>
                <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>

                <dt>DATE</dt>
                <dd>{ hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }</dd>

                <!-- Add: Time, author, lines edited -->
                <dt>TIME</dt>
                <dd>{ hoveredCommit.time?.slice(0,5) }</dd>

                <dt>AUTHOR</dt>
                <dd>{ hoveredCommit.author }</dd>

                <dt>LINES EDITED</dt>
                <dd>{ hoveredCommit.totalLines }</dd>
        </dl>
    </div>
    <BarHorizontal data={locBarData} title={title}/>
    <LineChart data={linesByDate}/>
</div>

<style>
    .charts {
        margin: 0;
        display: flex;
        flex-direction: column;
    }

    .toolplot {
        /* grid-template-columns: 1fr 200px; */
        margin-bottom: 2em;
    }

    .scattertitle {
        text-align: center;
        font-size: 120%;
    }

    .scatterplot {
        margin: 0;
    } 

	svg {
		overflow: visible;
        height: 60%;
        width: 100%;
        margin: 0em;
	}

    .gridlines {
        stroke-opacity: .2;
    }

    circle {
        fill-opacity: 70%;
        transition: 200ms;
        &:hover {
            fill:darkgreen;
        }
    }

    dl.info {
        display: grid; 

        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
    }

    .tooltip {
        position: fixed;

        /* top: 3em;
        left: 2em;
        grid-column: 1; */
        padding: 1em;

        box-shadow: 0 2px 10px grey;
        border-radius: 2px;
        background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
        backdrop-filter: blur(10px);
    }

    dt {
        grid-column: 1;
        margin: 0;
        color: grey;
        font-size: 75%;
    }

    dd {
        grid-column: 2;
        margin-left: 1em;
        font-size: 80%;
    }

    .selected {
        fill: red;
    }
</style>