<script>
	import * as d3 from "d3";
	import { onMount } from "svelte";

	let data = [];
	let commits = [];

	onMount(async () => {
		data = await d3.csv("loc.csv", row => ({
			...row,
			line: Number(row.line), // or just +row.line
			depth: Number(row.depth),
			length: Number(row.length),
			date: new Date(row.date + "T00:00" + row.timezone),
			datetime: new Date(row.datetime)
		}));
		// console.log(data)

		commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
			let first = lines[0];
			let {author, date, time, timezone, datetime} = first;
			let ret = {
				id: commit,
				url: "https://github.com/vis-society/lab-7/commit/" + commit,
				author, date, time, timezone, datetime,
				hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
				totalLines: lines.length
			};

			// Like ret.lines = lines
			// but non-enumerable so it doesn’t show up in JSON.stringify
			Object.defineProperty(ret, "lines", {
				value: lines,
				configurable: true,
				writable: true,
				enumerable: false,
			});

			return ret;
		});

		// console.log(commits) 

	});

	$: numDays = d3.groups(data, d => d.date).length;

	$: fileLengths = d3.rollups(data, v => d3.max(v, v => v.line), d => d.file);
	$: averageFileLength = Math.round(d3.mean(fileLengths, d => d[1]));

	$: workByPeriod = d3.rollups(data, v => v.length, d => d.datetime.toLocaleString("en", {dayPeriod: "short"}));
	$: maxPeriod = d3.greatest(workByPeriod, (d) => d[1])?.[0];


	// scatterplot

	let width = 1000, height = 600;
	
	let margin = {top: 10, right: 10, bottom: 30, left: 20};

	let usableArea = {
		top: margin.top,
		right: width - margin.right,
		bottom: height - margin.bottom,
		left: margin.left
	};
	usableArea.width = usableArea.right - usableArea.left;
	usableArea.height = usableArea.bottom - usableArea.top;

	// $: console.log(d3.extent(data, (d) => d.datetime));
	$: xScale = d3.scaleTime(d3.extent(data, (d) => d.datetime), [usableArea.left, usableArea.right]).nice();

	$: yScale = d3.scaleLinear()
		.domain([0, 24])
		.range([usableArea.bottom, usableArea.top]);

	let xAxis, yAxis;

	$: {
		d3.select(xAxis).call(d3.axisBottom(xScale));
		d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
	}

	let yAxisGridlines;

	$: {
		d3.select(yAxisGridlines).call(d3.axisLeft(yScale).tickFormat("").tickSize(-usableArea.width));
	}

	let hoveredIndex = -1;
	$: hoveredCommit = commits[hoveredIndex] ?? {};

	let cursor = {x: 0, y: 0};


</script>

<svelte:head>
	<title>Meta</title>
</svelte:head>
<h1>Meta</h1>
<p>This page includes stats about the code of this website.</p>

<h2>Summary</h2>
<dl class="stats">
	<dt>Total <abbr title="Lines of code">LOC</abbr></dt>
	<dd>{data.length}</dd>

	<dt>Total <abbr title="Commits">Commits</abbr></dt>
	<dd>{commits.length}</dd>

	<dt>Total <abbr title="Days worked on site">Days of Work</abbr></dt>
	<dd>{numDays}</dd>

	<dt>Average <abbr title="File length">Lines Per File</abbr></dt>
	<dd>{averageFileLength}</dd>

	<dt>Most Common <abbr title="Work Time">Work Time</abbr></dt>
	<dd>{maxPeriod}</dd>
</dl>

<h2>Commits by Time of Day</h2>
<svg viewBox="0 0 {width} {height}">
	<g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
	<g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
	<g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
	<g class="dots">
		{#each commits as commit, index }
			<circle
				cx={ xScale(commit.datetime) }
				cy={ yScale(commit.hourFrac) }
				r="5"
				fill="steelblue"
				on:mouseenter={evt => {
					hoveredIndex = index;
					cursor = {x: evt.x, y: evt.y};
				}}
				on:mouseleave={evt => hoveredIndex = -1}
				on:mouseenter={evt => {
		hoveredIndex = index;
		cursor = {x: evt.x, y: evt.y};
	}}
			/>
		{/each}
	</g>	
</svg>
<dl id="commit-tooltip" class="info tooltip" hidden={hoveredIndex === -1} style="top: {cursor.y}px; left: {cursor.x}px">
	<dt>Commit</dt>
	<dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>

	<dt>Date</dt>
	<dd>{ hoveredCommit.datetime?.toLocaleDateString("en", {date: "full"}) }</dd>

	<dt>Time</dt>
	<dd>{ hoveredCommit.datetime?.toLocaleTimeString() }</dd>

	<dt>Author</dt>
	<dd>{ hoveredCommit.author }</dd>

	<dt>Lines Edited</dt>
	<dd>{ hoveredCommit.totalLines }</dd>
</dl>


<style>
	svg {
		overflow: visible;
	}

	.gridlines {
		stroke-opacity: .2;
	}

	dl.info {
		display: grid;
		grid-template-columns: 40% 120%;
		dt {
			color: #5c5c5c;
		}
		transition-duration: 500ms;
		transition-property: opacity, visibility;

		&[hidden]:not(:hover, :focus-within) {
			opacity: 0;
			visibility: hidden;
		}
	}

	.tooltip {
		position: fixed;
		top: 1em;
		background-color: #f5f5f5;
		box-shadow: 5px 5px 5px #e6e6e6; 
		border-radius: 5px;
		padding: 1em;
	}

	circle {
		transition: 200ms;
		transform-origin: center;
		transform-box: fill-box;

		&:hover {
			transform: scale(1.5);
		}
	}


</style>
