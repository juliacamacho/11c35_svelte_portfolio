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
<section>
    <h2>My Projects</h2>
    <div class="projects">
        {#each projects.slice(0, 3) as p}
            <Project info={p} hLevel=3 />
        {/each}
    </div>
</section>
<section>
    <h2>My Github Stats</h2>
    {#await fetch("https://api.github.com/users/juliacamacho") }
        <p>Loading...</p>
    {:then response}
        {#await response.json()}
            <p>Decoding...</p>
        {:then data}
            <dl>
                <dt>Followers:</dt>
                <dd>{data.followers}</dd>
                <dt>Number of public repos:</dt>
                <dd>{data.public_repos}</dd>
            </dl>
        {:catch error}
            <p class="error">
                Something went wrong: {error.message}
            </p>
        {/await}
    {:catch error}
        <p class="error">
            Something went wrong: {error.message}
        </p>
    {/await}
</section>

<style>
    h2 {
	    font-size: 300%;
    }

    dl {
        display: grid;
        grid-template-columns: 200px 60px;
    }
</style>