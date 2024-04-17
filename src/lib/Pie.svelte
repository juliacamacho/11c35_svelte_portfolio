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

    function getEmptyArc (label, data = pieData) {
        // Union of old and new labels in the order they appear
        let labels = d3.sort(new Set([...oldData, ...pieData].map(d => d.label)));
        let labelIndex = labels.indexOf(label);
        let sibling;
        for (let i = labelIndex - 1; i >= 0; i--) {
            sibling = data.find(d => d.label === labels[i]);
            if (sibling) {
                break;
            }
        }

        let angle = sibling?.endAngle ?? 0;
        return {startAngle: angle, endAngle: angle};
    }

    function transitionArcs() {
        let wedgeElements = Object.values(wedges);

        d3.selectAll(wedgeElements).transition("arc")
            .duration(transitionDuration)
            .styleTween("d", function (_, index) {
                let wedge = this;
                let label = Object.keys(wedges)[index];

                let transition = transitionArc(wedge, label);
                return transition?.interpolator;
                
            });
    }

    function sameArc(arc_1, arc_2) {
        if ((!arc_1 && !arc_2) || (arc_1.startAngle == arc_2.startAngle && arc_1.endAngle == arc_2.endAngle)) {
            return true;
        }
    }
    
    function transitionArc (wedge, label) {
        label ??= Object.entries(wedges).find(([label, w]) => w === wedge)[0];

        let d = pieData.find(d => d.label === label);
        let d_old = oldData.find(d => d.label === label);

        if (sameArc(d_old, d)) {
            return null;
        }

        let from = d_old ? {...d_old} : getEmptyArc(label, oldData);
        let to = d ? {...d} : getEmptyArc(label, data);

        let angleInterpolator = d3.interpolate(from, to);
        let interpolator = t => `path("${ arcGenerator(angleInterpolator(t)) }")`;
        
        let type = d ? d_old ? "update" : "in" : "out";
        
        return {d, d_old, from, to, interpolator, type};
    }

    function arc (wedge) {
        // Calculations that will only be done once per element go here
        return {
            duration: transitionDuration,
            css: (t, u) => {
                // t is a number between 0 and 1 that represents the transition progress; u is 1 - t
                // TODO return CSS to be applied for the current t as a string
            }
        }
    }


</script>
<div class="container">
	<svg viewBox="-50 -50 100 100">
        {#each pieData as d, index (d.label)}
            <path d={d.arc} fill={ colors(d.id ?? d.label) }
                class:selected={selectedIndex === index}
                on:click={e => selectedIndex = selectedIndex === index ? -1 : index} 
                bind:this={ wedges[d.label] } 
                transition:arc />
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