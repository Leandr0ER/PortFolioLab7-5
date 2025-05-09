<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';

    import Bar from '$lib/Bar.svelte';

    let data = [];
    let commits = [];
    let width = 1000, height = 600;
    let margin = {top: 10, right: 10, bottom: 30, left: 20};
    let xAxis, yAxis;
    let yAxisGridlines;
    let hoveredIndex = -1;
    let cursor = {x: 0, y: 0};

    let commitProgress = 100;
    // MOVER ESTA DEFINICIÓN PARA QUE NO DEPENDA DE minDate/maxDate INICIALMENTE
    $: timeScale = d3.scaleTime().range([0, 100]);
    $: commitMaxTime = timeScale.invert(commitProgress);

    $: filteredCommits = commits.filter(commit => commit.datetime <= commitMaxTime);
    $: filteredLines = data.filter(line => new Date(line.datetime) <= commitMaxTime);

    // CALCULAR minDate y maxDate DESPUÉS DE FILTRAR LOS COMMITS
    $: minDate = filteredCommits.length > 0 ? d3.min(filteredCommits.map(d => d.date)) : new Date();
    $: maxDate = filteredCommits.length > 0 ? d3.max(filteredCommits.map(d => d.date)) : new Date();
    $: maxDatePlusOne = new Date(maxDate);
    $: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

    // ACTUALIZAR EL DOMINIO DE timeScale CUANDO minDate o maxDate CAMBIEN
    $: if (minDate && maxDate) {
        timeScale.domain([minDate, maxDate]);
    }

    $: xScale = d3.scaleTime()
        .domain([minDate, maxDatePlusOne])
        .range([usableArea.left, usableArea.right])
        .nice();

    $: yScale = d3.scaleLinear()
        .domain([24, 0])
        .range([usableArea.bottom, usableArea.top]);

    $: hoveredCommit = filteredCommits[hoveredIndex] ?? hoveredCommit ?? {};

    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;

    let commitTooltip;

    let tooltipPosition = {x: 0, y: 0};

    let clickedCommits = [];

    async function dotInteraction (index, evt) {
        let hoveredDot = evt.target;
        const commit = filteredCommits[index];
        if (evt.type === "mouseenter") {
            hoveredIndex = index;
            cursor = {x: evt.x, y: evt.y};
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: "fixed",
                middleware: [
                    offset(5),
                    autoPlacement()
                ],
            });
        }
        else if (evt.type === "mouseleave") {
            hoveredIndex = -1
        }
        else if (evt.type === "click") {
        if (!clickedCommits.includes(commit)) {
            clickedCommits = [...clickedCommits, commit];
        }
        else {
                clickedCommits = clickedCommits.filter(c => c !== commit);
        }
    }

    }

    onMount(async () => {
        data = await d3.csv("./loc.csv", row => ({
            ...row,
            line: Number(row.line),
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));

        commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
            let first = lines[0];
            let {author, date, time, timezone, datetime} = first;
            let ret = {
                id: commit,
                url: "https://github.com/Leandr0ER/PortFolioLab7-5/commit" + commit,
                author, date, time, timezone, datetime,
                hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                totalLines: lines.length
            };

            Object.defineProperty(ret, "lines", {
                value: lines,
                configurable: true,
                writable: true,
                enumerable: false,
            });

            return ret;
        });
        commits = d3.sort(commits, d => -d.totalLines);
    });



    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
    }

    $: {
        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
                .tickFormat("")
                .tickSize(-usableArea.width)
        );
    }

    $: rScale = d3.scaleSqrt()
        .domain(d3.extent(filteredCommits.map(d=>d.totalLines)))
        .range([2, 30]);

    $: allTypes = Array.from(new Set(filteredLines.map(d => d.type)));

    $: selectedLines = (clickedCommits.length > 0 ? clickedCommits : filteredCommits).flatMap(d => d.lines);

    $: selectedCounts = d3.rollup(
        selectedLines,
        v => v.length,
        d => d.type
    );

    $: languageBreakdown = allTypes.map(type => [type, selectedCounts.get(type) || 0]);

    $: totalLOC = filteredLines.length;
    $: totalFiles = d3.groups(filteredLines, d => d.file).length;
    $: totalCommits = filteredCommits.length;

</script>
<svelte:head>
<title>Meta</title>
</svelte:head>
<h1>Meta</h1>
<p>This page includes stats about the code of this website</p>
<p>Total lines of code: {totalLOC}</p>

<dl class="info tooltip" bind:this={commitTooltip} hidden={hoveredIndex === -1} style="top: {cursor.y}px; left: {cursor.x}px">
    <dt>Commit</dt>
    <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>

    <dt>Date</dt>
    <dd>{ hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }</dd>

    <dt>Author</dt>
    <dd>{ hoveredCommit.author }</dd>

    <dt>Time</dt>
    <dd>{ hoveredCommit.time }</dd>
</dl>

<p>Please adjust the slider to display the information.</p>
<div class="slider-container">
    <div class="slider">
        <label for="slider">Show commits until:</label>
        <input type="range" id="slider" name="slider" min=0 max=100 bind:value={commitProgress}/>
    </div>
    <time class="time-label">{commitMaxTime.toLocaleString()}</time>
</div>


<svg viewBox="0 0 {width} {height}">
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
    <g class="dots">
        {#each filteredCommits as commit, index (commit.id) }
        <circle
            class:selected={ clickedCommits.includes(commit) }
            on:click={ evt => dotInteraction(index, evt) }
            on:mouseenter={evt => dotInteraction(index, evt)}
            on:mouseleave={evt => dotInteraction(index, evt)}
            cx={ xScale(commit.datetime) }
            cy={ yScale(commit.hourFrac) }
            r={ rScale(commit.totalLines) }
            fill="steelblue"
            fill-opacity="0.5"
            style="--r: {rScale(commit.totalLines)};"
            />
        {/each}
    </g>
</svg>
<Bar data={languageBreakdown} width={width} />

<section>
    <h2>Summary</h2>
    <dl class="stats">
    <dt>Total <abbr title="Lines of code">LOC</abbr></dt>
    <dd>{totalLOC}</dd>
    <dt>Files</dt>
    <dd>{totalFiles}</dd>
    <dt>Commits</dt>
    <dd>{totalCommits}</dd>
    </dl>

</section>

<style>
    dl{
        display: grid;
        grid-template-columns: auto;
    }
    dt{
        grid-row: 1;
        font-family: inherit;
        font-weight: bold;
        color: var(--border-gray);
        text-transform: uppercase;
    }
    dd{
        font-family: inherit;
        font-weight: bold;
    }
    section{
        border-width:0.15em;
        border-style:solid;
        border-color:var(--border-gray);
        padding-left: 1em;
        padding-right: 1em;
    }
    svg {
        overflow: visible;
    }
    .gridlines {
        stroke-opacity: .2;
    }

.info{
    display: grid;
    margin:0;
    grid-template-columns: 2;
    background-color: oklch(100% 0% 0 / 80%);
    box-shadow: 1px 1px 3px 3px gray;
    border-radius: 5px;
    backdrop-filter: blur(10px);
    padding:10px;

    transition-duration: 200ms; /* MODIFIED HERE */
    transition-property: opacity, visibility, r; /* MODIFIED HERE */

    &[hidden]:not(:hover, :focus-within) {
        opacity: 0;
        visibility: hidden;
    }
}


.info dt{
    grid-column:1;
    grid-row:auto;
}

.info dd{
    grid-column:2;
    grid-row:auto;
    font-weight: 400;
}

.tooltip{
    position: fixed;
    top: 1em;
    left: 1em;
}

circle {
    transition: all 200ms, /* MODIFIED HERE */
                r calc(var(--r) * 100ms); /* MODIFIED HERE */
    transform-origin: center;
    transform-box: fill-box;

    @starting-style { /* ADD THIS BLOCK */
        r: 0;
    }

    &:hover {
        transform: scale(1.5);
    }
}

.selected {
    fill: var(--color-accent);
}

/* Lab 7.5 */
.slider-container{
    display:grid;
}
.slider{
    display: flex;
}
#slider{
    flex:1;
}
.time-label{
    font-size: 0.75em;
    text-align: right;
}

</style>