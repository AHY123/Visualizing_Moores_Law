<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    export let curtime;
    export let count;
    export let index;

    let gx, gy;
    let height = 30;
    let width = 400;
    let svg, text;

    let isVisible = true;
    $: if (curtime >= 3) {
        isVisible = true;
    } else {
        isVisible = false;
    }

    onMount(() => {
        svg = d3.selectAll("svg");

        text = svg.selectAll("text");

        function repeat() {
            if (index == count - 1) {
                text.text('~~~~~~~~ THE END! ~~~~~~~~')
            }
            else {
                text.text('SCROLL DOWN TO CONTINUE ...')
            }
            text.attr("opacity", 1)
                .transition()
                .duration(500)
                .ease(d3.easePoly.exponent(3))
                .attr("opacity", 0)
                .transition()
                .duration(500)
                .ease(d3.easePoly.exponent(3))
                .attr("opacity", 1)
                .on("end", repeat);
        }
        repeat();
    });
    
</script>

<div class="ScrollToContinueIcon" class:visible={isVisible}>
    <svg {width} {height} viewBox="0 0 {width} {height}">
        <text x={30} y={20} font-size="24px">
            SCROLL DOWN TO CONTINUE ...
        </text>
    </svg>
</div>

<style>
    .ScrollToContinueIcon {
        visibility: hidden;
    }

    .ScrollToContinueIcon.visible {
        visibility: visible;
    }
</style>
