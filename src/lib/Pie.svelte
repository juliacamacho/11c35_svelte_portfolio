<script>
    import * as d3 from 'd3';

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);

    export let data = [];

    let sliceGenerator = d3.pie().value(d => d.value);
    let arcData;
    let arcs;
    $: {
        arcData = sliceGenerator(data);
        arcs = arcData.map(d => arcGenerator(d));
    }

    let colors = d3.scaleOrdinal(d3.schemeTableau10);

</script>
<div class="container">
	<svg viewBox="-50 -50 100 100">
        {#each arcs as arc, i}
            <path d={ arc } fill={ colors(i) } />
        {/each}
    </svg>
    <ul class="legend">
        {#each data as d, index}
            <li style="--color: { colors(index) }">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>     


<style>
    div.container {
        display: flex;
        gap: 5em;
    }

    svg {
        max-width: 20em;
        margin-block: 2em;

        /* Do not clip shapes outside the viewBox */
        overflow: visible;
    }

    li {
        display: flex;
        align-items: center;
        gap: 1em;
    }

    ul.legend {
        flex: 1;
        list-style-type: none;
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
        border-style: solid;
        border-color: #c9c9c9;
        padding: 1em;
    }

    span.swatch {
        display: inline-block;
        width: 1em;
        height: 1em;
        background-color: var(--color);
    }
</style>