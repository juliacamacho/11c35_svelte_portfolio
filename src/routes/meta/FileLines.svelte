<script>
    import * as d3 from 'd3';
    import { flip as originalFlip } from "svelte/animate";

    export let lines = []

    let files = [];
    $: {
        files = d3.groups(lines, d => d.file).map(([name, lines]) => {
            return {name, lines};
        });
        files = d3.sort(files, d => -d.lines.length);
    }

    export let colors = d3.scaleOrdinal(d3.schemeTableau10);

    function getFlip () {
        return originalFlip;
    }
    $: flip = getFlip(files);

</script>

<dl class="files">
	{#each files as file (file.name) }
		<div animate:flip>
			<dt>
				<code>{file.name}</code>
                <small>{file.lines.length} lines</small>
			</dt>
            <dd>
                {#each file.lines as line (line.line) }
                    <div class="line" style="background: { colors(line.type) }" in:scale>
                    </div>
                {/each}
            </dd>
		</div>
	{/each}
</dl>

<style>
    small {
        display: block;
    }

    dl {
        display: grid;
        grid-template-columns: 30% auto;
        
        & > div {
            grid-column: 1 / -1;
            display: grid;
            grid-template-columns: subgrid;
            background: hsl(0 0% 100% / 90%);
            box-shadow: 0 0 .2em .2em hsl(0 0% 100% / 90%);
        }

        dt {
            grid-column: 1;
        }

        dd {
            grid-column: 2;
            display: flex;
            flex-wrap: wrap;
            align-items: start;
            align-content: start;
            gap: .15em;
            padding-top: .6em;
            margin-left: 0;
        }
    }

    .line {
        display: flex;
        width: .5em;
        aspect-ratio: 1;
        /* background: steelblue; */
        border-radius: 50%;
    }

</style>