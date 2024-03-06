<script>
    import projects from '$lib/projects.json';
    import Project from "$lib/Project.svelte";
</script>
<svelte:head>
	<title>Home</title>
</svelte:head>
<h1>Julia Camacho</h1>
<p>Hi! I'm a junior majoring in Course 11-6 (Urban Science and Planning with Computer Science).</p>
<img src="images/sf_headshot.jpg" class="home" alt="A photo of myself standing against a body of water in San Francisco.">
<h2>My Projects</h2>
<div class="projects">
    {#each projects.slice(0, 3) as p}
        <Project info={p} hLevel=3 />
    {/each}
</div>

<style>
    h2 {
	    font-size: 300%;
    }
</style>