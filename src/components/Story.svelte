<script>
    // imports
    import { onMount } from "svelte";
    import text_data from "./text_data.json";
    import milestone_data from "./milestone_data.json";
    import * as d3 from "d3";
    import Scroller from "@sveltejs/svelte-scroller";
    import ScrollToContinueIcon from "./small_components/ScrollToContinueIcon.svelte";
    import TitleScreen from "./TitleScreen.svelte";
    import WriteUpScreen from "./WriteUpScreen.svelte";
    import StoryGraph from "./StoryGraph.svelte";
    import GenericLabelText from "./GenericLabelText.svelte";
    import ComputingCostGraph from "./ComputingCostGraph.svelte";
    import GenericSubtitle from "./GenericSubtitle.svelte";
    import GenericText from "./GenericText.svelte";
    import GuessGame from "./GuessGame.svelte";

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
        data = await d3.csv("transistors_2.csv", (d) => {
            return {
                date: new Date(d.Year),
                value: +d["transistors_per_microprocessor"],
                expected_value: +d["expected_transistors"],
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
        "subtitle",
        "text",
        "text",
        "text",
        "milestones",
        -1,
        "text",
        "text",
        -1,
        "linear",
        -1,
        "subtitle",
        "text",
        "text",
        "guess",
        "text",
        "cost",
        "text",
        "text",
        -1,
        "subtitle",
        10,
    ];
    // for programmer use
    const section_dict = [
        "section",
        "gap",
        "linear",
        "milestones",
        "log",
        "cost",
        "subtitle",
        "text",
        "guess",
    ];
    // index to text pairs
    const section_text = text_data;
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
            // } else if (section_settings[i] == "linear") {
            //     sections.push([section_count, "linear"]);
            //     section_count++;
            // } else if (section_settings[i] == "log") {
            //     sections.push([section_count, "log"]);
            //     section_count++;
            // } else if (section_settings[i] == "milestones") {
            //     sections.push([section_count, "milestones"]);
            //     section_count++;
            // } else if (section_settings[i] == "cost") {
            //     sections.push([section_count, "cost"]);
            //     section_count++;
            // } else if (section_settings[i] == "subtitle") {
            //     sections.push([section_count, "subtitle"]);
            //     section_count++;
            // } else if (section_settings[i] == "text") {
            //     sections.push([section_count, "text"]);
            //     section_count++;
        } else {
            sections.push([section_count, section_settings[i]]);
            section_count++;
        }
        sections_fade.push(false);
    }

    $: {
        for (let i = 0; i < sections_fade.length; i++) {
            if (i < index) {
                sections_fade[i] = true;
            } else {
                sections_fade[i] = false;
            }
        }
    }

    let left_title = "";
    let left_text = "";
    let right_title = "";
    let right_text = "";
    let milestone_index = 0;
    $: {
        if (section_settings[index] == "milestones") {
            left_title = "Milestone Explorer";
            left_text =
                "Use the buttons to explore the data and learn about the 10 milestones!";
            right_title = Object.keys(milestone_data)[milestone_index];
            right_text =
                "Milestone " +
                (milestone_index + 1) +
                ": " +
                Object.values(milestone_data)[milestone_index];
        } else {
            left_title = "";
            left_text = "";
            right_title = "";
            right_text = "";
        }
    }
    $: {
        console.log("index: " + index);
    }
</script>

<main>
    <Scroller
        top={0.0}
        bottom={1}
        threshold={0.55}
        bind:count
        bind:index
        bind:offset
        bind:progress
    >
        <div class="foreground" slot="foreground">
            {#each sections as sec}
                <!-- <p>{sec[0]}{sec[1]}</p> -->
                {#if sec[1] == "section"}
                    <section class="snapper" class:fade={sections_fade[sec[0]]}>
                        <!-- <div class="content_sticky"></div> -->
                    </section>
                {:else if sec[1] == "gap"}
                    <section class="gap" class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky"></div>
                    </section>
                {:else if sec[1] == "linear"}
                    <section class="snapper" class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky">
                            <StoryGraph {data} display_mode="linear" />
                        </div>
                    </section>
                {:else if sec[1] == "log"}
                    <section class="snapper" class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky">
                            <StoryGraph {data} display_mode="log" />
                        </div>
                    </section>
                {:else if sec[1] == "milestones"}
                    <section class="snapper" class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky">
                            <StoryGraph
                                {data}
                                display_mode="linear"
                                is_milestones={true}
                                bind:milestone_index
                            />
                        </div>
                    </section>
                {:else if sec[1] == "cost"}
                    <section class="snapper" class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky">
                            <ComputingCostGraph {cost_data} />
                        </div>
                    </section>
                {:else if sec[1] == "subtitle"}
                    <section class="snapper" class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky">
                            <GenericSubtitle text={section_text[sec[0]]} />
                        </div>
                    </section>
                {:else if sec[1] == "text"}
                    <section class="snapper" class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky">
                            <GenericText text={section_text[sec[0]]} />
                        </div>
                    </section>
                {:else if sec[1] == "guess"}
                    <section class="snapper" class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky">
                            <GuessGame {cost_data} />
                        </div>
                    </section>
                {:else}
                    <section class:fade={sections_fade[sec[0]]}>
                        <div class="content_sticky"></div>
                    </section>
                {/if}
            {/each}
        </div>
        <div
            class="background"
            slot="background"
            bind:clientWidth={width}
            bind:clientHeight={height}
        >
            <div class="centering_foreground">
                {#if index == 0 || index == 1}
                    <TitleScreen {index} />
                {/if}
                {#if index >= 3 && index <= 10}
                    <GenericLabelText
                        {index}
                        fadeIn="3"
                        fadeOut="9"
                        title="Chapter 1: What Is Moore's Law"
                        {left_title}
                        left={left_text}
                        {right_title}
                        right={right_text}
                    />
                {/if}
                {#if index >= 10 && index <= 12}
                    <GenericLabelText
                        {index}
                        fadeIn="10"
                        fadeOut="11"
                        title="Reader Playground"
                        left_title={"Explore The Data"}
                        left={"Here we present an interactive visualization of Transistor dataset. Play with the time range to see if you can find anything interesting."}
                        right_title={"Make Your Own Conclusions"}
                        right={"Particularly on the Linear Scale, see if we have been following Moore's Law in recent years."}
                    />
                {/if}
                {#if index >= 13 && index <= 16}
                    <GenericLabelText
                        {index}
                        fadeIn="13"
                        fadeOut="17"
                        title="Chapter 2: Why Does Moore's Law Matter"
                    />
                {/if}
                {#if index >= 16 && index <= 17}
                    <GenericLabelText
                        {index}
                        fadeIn="16"
                        fadeOut="18"
                        title="Chapter 2: Why Does Moore's Law Matter"
                        left={"Interact with this graph to see how expensive computer components were."}
                        right={"Remember that this graph is in logarithmic scale so the price of components have indeed been exponentially decreasing."}
                    />
                {/if}
                {#if index >= 17 && index <= 21}
                    <GenericLabelText
                        {index}
                        fadeIn="17"
                        fadeOut="20"
                        title="Chapter 2: Why Does Moore's Law Matter"
                    />
                {/if}
                <!-- {#if index == 7 || 8}
                    <GenericLabelText
                        {index}
                        fadeIn="7"
                        fadeOut="8"
                        title="Demonstrating How Moore's Law Relates to Costs of Computing Storage"
                        left="You can see that the price of storage has decreased exponentially. We will explain why and how technological advances take exponential growth."
                        right="(This graph also uses a log scale)"
                    />
                {/if} -->
                <!-- {#if index == 10 || 11}
                    <GenericLabelText
                        {index}
                        fadeIn="10"
                        fadeOut="11"
                        title="Reader Playground"
                        left="Explore the data for yourself! Try different data and scales! For all of our graphs, we have tool tips and for this graph we have added date filters, options to change axis, and other interactions."
                        right="The most interesting (and hardest to implement) feature is the double sided slider for date filtering. It is a great feature because it allows readers to explore the data at different time frames. This is especially relevant because then readers could be able to tell that the data followed Moore's Law far better in the earlier years than the recent years."
                    />
                {/if} -->
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
        height: 100vh;
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
        height: 90vh;
        background-color: rgba(0, 0, 0, 0);
        color: black;
        text-align: center;
        max-width: 1000%;
        padding: 1em;
        margin: 0 0 2em 0;
        align-items: center;

        /* position: sticky;
        top: 0%; */

        animation: fadeIn 0.5s;
        animation-fill-mode: forwards;
    }
    section.snapper {
        scroll-snap-align: start;
        /* outline: magenta solid 3px; */
    }
    section.gap {
        height: 90vh;
    }
    section.fade {
        animation: fadeOut 0.5s;
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
    .centering_foreground {
        width: 100%;
        height: 92%;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-flow: column wrap;
    }
    .content_sticky {
        height: 100%;
        /* position: sticky;
        top: 0%; */
    }
</style>
