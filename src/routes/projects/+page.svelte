<script>
    import * as d3 from 'd3';

    import projects from '$lib/projects.json';
    import Project from "$lib/Project.svelte";
    import Pie from '$lib/Pie.svelte';

    let filteredProjects;
    $: filteredProjects = projects.filter(project => {
        let values = Object.values(project).join("\n").toLowerCase();
        return values.includes(query.toLowerCase());
    });


    let pieData
    $: {
        pieData = {};
        let rolledData = d3.rollups(filteredProjects, v => v.length, d => d.year);
        pieData = rolledData.map(([year, count]) => {
            return { value: count, label: year };
        });
    }

    let query = "";

</script>

<svelte:head>
	<title>Projects</title>
</svelte:head>
<h1>My {projects.length} Projects</h1>
<Pie data = {pieData}/>
<input type="search" bind:value={query}
       aria-label="Search projects" placeholder="🔍 Search projects…" />
<div class="projects">
    {#each filteredProjects as p}
        <Project info={p} />
    {/each}
</div>
