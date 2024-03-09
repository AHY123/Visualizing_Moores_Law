<script>
    // imports
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import Scroller from "@sveltejs/svelte-scroller";
    import ScrollToContinueIcon from "./ScrollToContinueIcon.svelte";
    import TitleScreen from "./TitleScreen.svelte";
    import WriteUpScreen from "./WriteUpScreen.svelte";

    // scoller var
    let count,
        index,
        offset,
        progress = 0;
    let width, height;

    // timer var
    let timer = null;
    let curTime = null;
    let startTime = null;

    // data var
    data = [];

    // collect data and start timer at mount
    onMount(async () => {
        startTimer();
        data = await d3.csv("transistors.csv", (d) => {
            return {
                date: new Date(d.Year),
                value: +d["transistors_per_microprocessor"],
            };
        });
        console.log(data);
    });

    // defining timer
    const startTimer = () => {
        startTime = Date.now();
        timer = setInterval(() => {
            const elapsedTimeInSeconds = Math.floor(
                (Date.now() - startTime) / 1000,
            );
            curTime = elapsedTimeInSeconds;
        }, 1000);
    };

    // reset timer when index changes
    $: {
        if (index) {
        }
        startTime = Date.now();
    }

    // index, section, visibility management
    const section_settings = [1, 'AIO', 3, 'GAP', 1];
</script>

<main>
    <Scroller
        top={0.0}
        bottom={1}

        
        threshold={0.8}
        bind:count
        bind:index
        bind:offset
        bind:progress
    >
        <div
            class="foreground"
            slot="foreground"
        >

        </div>
        <div
            class="background"
            slot="background"
            bind:clientWidth={width}
            bind:clientHeight={height}
        ></div>
    </Scroller>
</main>
