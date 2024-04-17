<script>
    import * as d3 from 'd3';

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
    let sliceGenerator = d3.pie().value(d => d.value).sort(null);

    export let data = [];
    let oldData = [];

    let pieData;
    $: {
        oldData = pieData;
        pieData = data.map(d => ({...d}));
        pieData = d3.sort(data, d => d.label);
        let arcData = sliceGenerator(pieData);
        let arcs = arcData.map(d => arcGenerator(d));
        pieData = pieData.map((d, i) => ({...d, ...arcData[i], arc: arcs[i]}));
        transitionArcs();
    };

    export let colors = d3.scaleOrdinal(d3.schemeTableau10);

    export let selectedIndex = -1;

    let wedges = {};
    $: console.log("WEDGES:", wedges);
    export let transitionDuration = 1000;

    function transitionArcs() {
        let wedgeElements = Object.values(wedges);

        d3.selectAll(wedgeElements).transition("arc")
            .duration(transitionDuration)
            .styleTween("d", function (_, index) {
                let wedge = this;
                let label = Object.keys(wedges)[index];
                let d = pieData.find(d => d.label === label);
                let d_old = oldData.find(d => d.label === label);
                if (!d || !d_old) {
                    return;
                }
                let from = {...d_old};
                let to = {...d};
                let angleInterpolator = d3.interpolate(from, to);
                let interpolator = t => `path("${ arcGenerator(angleInterpolator(t)) }")`;
                return interpolator;
            });
    }

</script>
<div class="container">
	<svg viewBox="-50 -50 100 100">
        {#each pieData as d, index (d.label)}
            <path d={d.arc} fill={ colors(d.id ?? d.label) }
                class:selected={selectedIndex === index}
                on:click={e => selectedIndex = selectedIndex === index ? -1 : index} 
                bind:this={ wedges[d.label] } />
        {/each}
    </svg>
    <ul class="legend">
        {#each pieData as d, index (d.label)}
            <li style="--color: { colors(d.label) }"                 class:selected={selectedIndex === index}>
                <span class="swatch"></span>
                {d.latrabel} <em>({d.value})</em>
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

    svg:has(path:hover) {
        path:not(:hover) {
            opacity: 50%;
        }
    }

    .selected {
        --color: oklch(60% 45% 0) !important;

        &:is(path) {
            fill: var(--color);
        }
    }

    path {
        transition: 300ms;
        /* transition-property: transform, opacity, fill; */
        fill-opacity: 75%
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