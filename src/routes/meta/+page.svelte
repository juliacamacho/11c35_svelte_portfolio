<script>
	import * as d3 from "d3";
	import Scrolly from "svelte-scrolly";
	import { onMount } from "svelte";
	import Pie from "$lib/Pie.svelte";
	import FileLines from "./FileLines.svelte"
	import {
		computePosition,
		autoPlacement,
		offset,
	} from '@floating-ui/dom';

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

		// console.log(commits);

	});

	// Filtering by time
	let commitProgress = 100;
	$: timeScale = d3.scaleTime(d3.extent(data, (d) => d.datetime), [0, 100]).nice();
	$: commitMaxTime = timeScale.invert(commitProgress);

	$: filteredCommits = commits.filter(commit => commit.datetime <= commitMaxTime)
	$: filteredLines = data.filter(line => line.datetime <= commitMaxTime)

	let filesCommitProgress = 100;
	$: filesTimeScale = d3.scaleTime(d3.extent(data, (d) => d.datetime), [0, 100]).nice();
	$: filesCommitMaxTime = filesTimeScale.invert(filesCommitProgress);

	$: filesFilteredCommits = commits.filter(commit => commit.datetime <= filesCommitMaxTime)
	$: filesFilteredLines = data.filter(line => line.datetime <= filesCommitMaxTime)


	// Summary stats
	$: numDays = d3.groups(filteredLines, d => d.date).length;

	$: fileLengths = d3.rollups(filteredLines, v => d3.max(v, v => v.line), d => d.file);
	$: averageFileLength = Math.round(d3.mean(fileLengths, d => d[1]));

	$: workByPeriod = d3.rollups(filteredLines, v => v.length, d => d.datetime.toLocaleString("en", {dayPeriod: "short"}));
	$: maxPeriod = d3.greatest(workByPeriod, (d) => d[1])?.[0];


	// Scatterplot setup

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
	$: xScale = d3.scaleTime(d3.extent(filteredLines, (d) => d.datetime), [usableArea.left, usableArea.right]).nice();

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

	let hoveredCommit = [];
	let hoveredIndex = -1;
	$: {
		if (hoveredIndex !== -1 ) {
			hoveredCommit = filteredCommits[hoveredIndex] ?? {};
		} 
	}
	let svg;

	// Selecting commits by brushing

	function brushed (evt) {
		let brushSelection = evt.selection;
		selectedCommits = !brushSelection ? [] : commits.filter(commit => {
			let min = {x: brushSelection[0][0], y: brushSelection[0][1]};
			let max = {x: brushSelection[1][0], y: brushSelection[1][1]};
			let x = xScale(commit.date);
			let y = yScale(commit.hourFrac);

			return x >= min.x && x <= max.x && y >= min.y && y <= max.y;
		});
	}

	function isCommitSelected (commit) {
		return selectedCommits.includes(commit);
	}

	$: {
		d3.select(svg).call(d3.brush().on("start brush end", brushed));
		d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
	}

	let selectedCommits = [];
	$: hasSelection = selectedCommits.length > 0;

	$: selectedLines = (hasSelection ? selectedCommits : commits).flatMap(d => d.lines);
	$: languageBreakdown =  d3.rollups(selectedLines, v => v.length, d => d.type);

	let commitTooltip;
	let tooltipPosition = {x: 0, y: 0};

	async function dotInteraction (index, evt) {
		let hoveredDot = evt.target;
			
		if (evt.type === "mouseenter" || evt.type === "focus") {
			hoveredIndex = index;
			tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
				strategy: "fixed", // because we use position: fixed
				middleware: [
					offset(10), // spacing from tooltip to dot
					autoPlacement() // see https://floating-ui.com/docs/autoplacement
				],
			});
		}
		else if (evt.type === "mouseleave" || evt.type === "blur") {
			hoveredIndex = -1
		}
		else if (evt.type === "click" || (evt.type = "keyup" && evt.key == "Enter")) {
			selectedCommits = [commits[index]];
			// console.log("selectedCommits:", selectedCommits);
		}
	}

	let colors = d3.scaleOrdinal(d3.schemeTableau10);

	let commitMessages = ["established my website", "created my projects page", "added my CV and contact page", "improved my projects page",
						"improved my contact page", "added a pie chart", "added projects to my projects page", "improved my pie chart", "improved my home page",
						"improved my home page again by adding statistics", "added a meta page", "improed the meta page by adding dynamic statistics",
						"improved my meta page by adding visualizations", "improved my meta page by adding scrolling"]

</script>


<svelte:head>
	<title>Meta</title>
</svelte:head>
<h1>Meta</h1>
<p>This page includes stats about the code of this website.</p>

<h2>Summary</h2>

<!-- <label>
	<input type=range bind:value={commitProgress}/>
	<time>{commitMaxTime.toLocaleString("en", {dateStyle: "long", timeStyle: "short"})}</time>
</label> -->

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

<Scrolly bind:progress={ commitProgress }>
	{#each commits as commit, index }
		<p>
			On {commit.datetime.toLocaleString("en", {dateStyle: "full", timeStyle: "short"})},
			I made <a href="{commit.url}" target="_blank">{ index > 0 ? 'another commit' : 'my first commit' }</a>.
			I edited {commit.totalLines} lines across { d3.rollups(commit.lines, D => D.length, d => d.file).length } files.
			In this commit, I {index < commitMessages.length ? commitMessages[index] : "made more improvements"}.
		</p>
	{/each}
	<svelte:fragment slot="viz">
		<h2>Commits by Time of Day</h2>
		<svg viewBox="0 0 {width} {height}" bind:this={svg}>
			<g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
			<g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
			<g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
			<g class="dots">
				{#each filteredCommits as commit, index (commit.id) }
					<circle
						cx={ xScale(commit.datetime) }
						cy={ yScale(commit.hourFrac) }
						r="5"
						fill="steelblue"
						tabindex="0"
						aria-describedby="commit-tooltip"
						aria-haspopup="true"
						class:selected={isCommitSelected(commit)}
						on:mouseenter={evt => dotInteraction(index, evt)}
						on:mouseleave={evt => dotInteraction(index, evt)}
						on:focus={evt => dotInteraction(index, evt)}
						on:blur={evt => dotInteraction(index, evt)}
						on:click={evt => dotInteraction(index, evt)}
						on:keyup={evt => dotInteraction(index, evt)}
					/>
				{/each}
			</g>	
		</svg>

		<!-- tooltip -->
		<dl id="commit-tooltip" class="info tooltip" role="tooltip" bind:this={commitTooltip} hidden={hoveredIndex === -1} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px">
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

		<p>{hasSelection ? selectedCommits.length : "No"} commits selected</p>
		<div class="language">
		{#each languageBreakdown as [language, lines] }
			<div>
				<p class="language-title">{language}</p>
				<p>{lines} lines ({d3.format(".2f")((lines / selectedLines.length) * 100)}%)</p>
			</div>
		{/each}
		</div>
		<Pie data={Array.from(languageBreakdown).map(([language, lines]) => ({label: language, value: lines}))} colors={colors}/>
	</svelte:fragment>
</Scrolly>

<div style="margin-top: 10em"></div>

<Scrolly bind:progress={ filesCommitProgress } --scrolly-layout="viz-first" --scrolly-viz-width="1.5fr">
	{#each commits as commit, index }
		<p>
			On {commit.datetime.toLocaleString("en", {dateStyle: "full", timeStyle: "short"})},
			I made <a href="{commit.url}" target="_blank">{ index > 0 ? 'another glorious commit' : 'my first commit, and it was glorious' }</a>.
			I edited {commit.totalLines} lines across { d3.rollups(commit.lines, D => D.length, d => d.file).length } files.
			In this commit, I {index < commitMessages.length ? commitMessages[index] : "made more improvements"}.
		</p>
	{/each}
	<svelte:fragment slot="viz">
		<FileLines lines={filesFilteredLines} colors={colors}/>
	</svelte:fragment>
</Scrolly>



<style>

	:global(body) {
		max-width: min(120ch, 80vw);
	}

	svg {
		overflow: visible;
	}

	.gridlines {
		stroke-opacity: .2;
	}

	.language-title {
		font-weight: bold;
	}

	.language {
		display: grid;
		grid-template-columns: 25% 25% 25% 25%;
	}

	dl.info {
		display: grid;
		grid-template-columns: 40% 120%;
		dt {
			color: #5c5c5c;
		}
		transition-duration: 500ms;
		/* transition-property: opacity, visibility; */

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

		@starting-style {
			r: 0;
		}

	}

	.selected {
		fill: red;
	}

</style>
