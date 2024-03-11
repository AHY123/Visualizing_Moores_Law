<script>
    // imports
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import Scroller from "@sveltejs/svelte-scroller";
    import ScrollToContinueIcon from "./small_components/ScrollToContinueIcon.svelte";
    import TitleScreen from "./TitleScreen.svelte";
    import WriteUpScreen from "./WriteUpScreen.svelte";
    import StoryGraph from "./StoryGraph.svelte";
    import GenericText from "./GenericText.svelte";
    import ComputingCostGraph from "./ComputingCostGraph.svelte";

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
    let cost_data = [];

    // collect data and start timer at mount
    onMount(async () => {
        startTimer();
        data = await d3.csv("transistors.csv", (d) => {
            return {
                date: new Date(d.Year),
                value: +d["transistors_per_microprocessor"],
            };
        });
        cost_data = await d3.csv("cost_of_computing.csv", (d) => {
            return {
                date: new Date(d.Year),
                memory: +d["memory"],
                flash: +d["flash"],
                drives: +d["disk_drives"],
                ssd: +d["ssd"],
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
    const section_settings = [
        1,
        "linear",
        -2,
        "log",
        -2,
        "cost",
        -2,
        "linear",
        -1,
        -1,
        1,
    ];
    const section_dict = ["section", "gap", "linear", "log", "cost"];
    let linear = "linear";
    let log = "log";
    let sections = [];
    let sections_fade = [];
    let section_count = 0;
    // console.log(sections)
    for (let i = 0; i < section_settings.length; i++) {
        // console.log(section_settings[i]);
        if (!isNaN(section_settings[i])) {
            for (let j = 0; j < Math.abs(section_settings[i]); j++) {
                if (section_settings[i] > 0) {
                    sections.push([section_count, "section"]);
                } else {
                    sections.push([section_count, "gap"]);
                }
                section_count++;
            }
        } else if (section_settings[i] == "linear") {
            sections.push([section_count, "linear"]);
            section_count++;
        } else if (section_settings[i] == "log") {
            sections.push([section_count, "log"]);
            section_count++;
        } else if (section_settings[i] == "cost") {
            sections.push([section_count, "cost"]);
            section_count++;
        } else {
            section_count++;
        }
        sections_fade.push(false);
    }

    $: {
        for (let i = 0; i < sections_fade.length; i++) {
            if (i + 1 <= index) {
                sections_fade[i] = true;
            } else {
                sections_fade[i] = false;
            }
        }
    }
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
                {#if sec[1] == "section"}
                    <section class:fade={sections_fade[sec[0]]}></section>
                {:else if sec[1] == "gap"}
                    <section
                        class="gap"
                        class:fade={sections_fade[sec[0]]}
                    ></section>
                {:else if sec[1] == "linear"}
                    <section class:fade={sections_fade[sec[0]]}>
                        <StoryGraph {data} display_mode="linear" />
                    </section>
                {:else if sec[1] == "log"}
                    <section class:fade={sections_fade[sec[0]]}>
                        <StoryGraph {data} display_mode="log" />
                    </section>
                {:else if sec[1] == "cost"}
                    <section class:fade={sections_fade[sec[0]]}>
                        <ComputingCostGraph {cost_data} {index} />
                    </section>
                {:else}
                    <section class:fade={sections_fade[sec[0]]}></section>
                {/if}
            {/each}
        </div>
        <div
            class="background"
            slot="background"
            bind:clientWidth={width}
            bind:clientHeight={height}
        >
            <div class="centering_container">
                {#if index == 0}
                    <TitleScreen {index} />
                {/if}
                {#if index == 1 || index == 2}
                    <GenericText
                        {index}
                        fadeIn="1"
                        fadeOut="2"
                        title="Starting With a Graph of Linear Scale"
                        left="Plotting data from 1970 to 2021, we can clearly see a exponential relationship."
                        right="We will be showing an expected Moore's Law exponential line for reference"
                    />
                {/if}
                {#if index == 4 || 5}
                    <GenericText
                        {index}
                        fadeIn="4"
                        fadeOut="5"
                        title="Presented using a Log Scale Graph"
                        left="We see that the points now form a straight line with using a log scale"
                        right="We will add an expected Moore's Law linear line for reference"
                    />
                {/if}
                {#if index == 7 || 8}
                    <GenericText
                        {index}
                        fadeIn="7"
                        fadeOut="8"
                        title="Demonstrating How Moore's Law Relates to Costs of Computing Storage"
                        left="You can see that the price of storage has decreased exponentially"
                        right="(This graph also uses a log scale)"
                    />
                {/if}
                {#if index == 10 || 11}
                    <GenericText
                        {index}
                        fadeIn="10"
                        fadeOut="11"
                        title="Reader Playground"
                        left="Explore the data for yourself! Try different data and scales! For all of our graphs, we have tool tips and for this graph we have added date filters, options to change axis, and other interactions."
                        right="The most interesting (and hardest to implement) feature is the double sided slider for date filtering. It is a great feature because it allows readers to explore the data at different time frames. This is especially relevant because then readers could be able to tell that the data followed Moore's Law far better in the earlier years than the recent years"
                    />
                {/if}
                {#if index == section_count - 1}
                    <WriteUpScreen />
                {/if}
            </div>
            <!-- <div class="progress-bars">
                <p>current section: <strong>{index + 1}/{count}</strong></p>
                <progress value={count ? (index + 1) / count : 0} />
                <p>offset in current section</p>
                <progress value={offset || 0} />
                <p>total progress</p>
                <progress value={progress || 0} />
            </div> -->
            <div class="scroll_down">
                <ScrollToContinueIcon {curTime} {count} {index} />
            </div>
        </div>
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
        display: flex;
        justify-content: center;
        align-items: center;
        flex-flow: column wrap;
    }
    section {
        height: 100vh;
        background-color: rgba(0, 0, 0, 0);
        color: white;
        text-align: center;
        max-width: 1000%;
        color: black;
        padding: 1em;
        margin: 0 0 2em 0;
        align-items: center;

        position: sticky;
        top: 10%;

        animation: fadeIn 1s;
        animation-fill-mode: forwards;
    }
    section.gap {
        height: 10vh;
    }
    section.fade {
        animation: fadeOut 1s;
        animation-fill-mode: forwards;
    }
    @keyframes fadeIn {
        0% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }
    @keyframes fadeOut {
        0% {
            opacity: 1;
        }
        100% {
            opacity: 0;
        }
    }
    .centering_container {
        width: 100%;
        height: 92%;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-flow: column wrap;
    }
</style>
