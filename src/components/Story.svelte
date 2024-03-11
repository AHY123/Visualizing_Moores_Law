<script>
    // imports
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import Scroller from "@sveltejs/svelte-scroller";
    import ScrollToContinueIcon from "./ScrollToContinueIcon.svelte";
    import TitleScreen from "./TitleScreen.svelte";
    import WriteUpScreen from "./WriteUpScreen.svelte";
    import StoryGraph from "./StoryGraph.svelte";

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
    let data = [];

    // collect data and start timer at mount
    onMount(async () => {
        startTimer();
        data = await d3.csv("transistors.csv", (d) => {
            return {
                date: new Date(d.Year),
                value: +d["transistors_per_microprocessor"],
            };
        });
        // console.log(data);
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
    const section_settings = [1, "AIO", 3, -1, 1];
    // const section_dict = ['section', 'gap', 'AIO']
    let sections = [];
    let section_count = 0;
    // console.log(sections)
    for (let i = 0; i < section_settings.length; i++) {
        // console.log(section_settings[i]);
        if (!isNaN(section_settings[i])){
            // console.log('is number')
            for (let j = 0; j < Math.abs(section_settings[i]); j++) {
                if (section_settings[i] > 0) {
                    // console.log([section_count, 'section']);
                    sections.push([section_count, 'section']);
                }
                else {
                    // console.log([section_count, 'gap']);
                    sections.push([section_count, 'gap']);
                }
                section_count ++
            }
        }
        else if (section_settings[i] == 'AIO') {
            // console.log('its a graph');
            // console.log([section_count, 'AIO']);
            sections.push([section_count, 'AIO']);
            section_count ++;
        }
        else {
            section_count ++;
        }
        // console.log(sections)
    }
    // console.log('Number of sections: ' + section_count)
    // console.log('sections:' + sections);
    // for (let i = 0; i < sections.length; i++) {
    //     console.log(sections[i]);
    // }
    // console.log(sections)
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
        <div class="foreground" slot="foreground">
            {#each sections as sec}
                <!-- <p>{sec[0]}{sec[1]}</p> -->
                {#if sec[1] == 'section'}
                            <section>{sec[0]}{sec[1]}</section>
                {:else if sec[1] == 'gap'}
                            <section class='gap'>{sec[0]}{sec[1]}</section>
                {:else if sec[1] == "AIO"}
                    <section>{sec[0]}{sec[1]}<StoryGraph {data}/></section>
                {:else}
                    <section>{sec[0]}{sec[1]}</section>
                {/if}
            {/each}
        </div>
        <div
            class="background"
            slot="background"
            bind:clientWidth={width}
            bind:clientHeight={height}
        ></div>
    </Scroller>
</main>

<style>
    @import url("https://fonts.googleapis.com/css2?family=Lato:ital,wght@0,100;0,300;0,400;1,100;1,300;1,400&display=swap");

    main {
        font-family: "Lato", sans-serif;
    }
    .foreground {
        width: 70%;
        margin: 0 auto;
        height: auto;
        position: relative;
    }
    .background {
        width: 100%;
        height: 100vh;
        position: relative;
    }
    section {
        height: 100vh;
        background-color: rgba(0, 0, 0, 0); /* 20% opaque */
        color: white;
        outline: magenta solid 3px;
        text-align: center;
        max-width: 1000%; /* adjust at will */
        color: black;
        padding: 1em;  
        margin: 0 0 2em 0;
        align-items: center;

        position: sticky;
        top: 10%;
    }
    section.gap {
        height: 10vh;
    }
</style>
