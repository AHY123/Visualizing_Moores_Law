<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import Graph from "./Graph.svelte";
  import LogGraph from "./LogGraph.svelte";
  import ComputingCostGraph from "./ComputingCostGraph.svelte";
  import ScrollToContinueIcon from "./ScrollToContinueIcon.svelte";
  import TitleScreen from "./TitleScreen.svelte";
  import WriteUpScreen from "./WriteUpScreen.svelte";
  // import StoryGraph from "./StoryGraph.svelte";
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let count = 0;
  let index = 0;
  let offset = 0;
  let progress = 0;
  let width, height;
  let timer = null;
  let curtime = null;
  let startTime = null;
  let data = [];
  let cost_data = [];

  const startTimer = () => {
    startTime = Date.now();
    timer = setInterval(() => {
      const elapsedTimeInSeconds = Math.floor((Date.now() - startTime) / 1000); // Calculate elapsed time in seconds
      // console.log('Elapsed Time:', elapsedTimeInSeconds, 'seconds');
      curtime = elapsedTimeInSeconds;
      // console.log(curtime)
    }, 1000);
  };

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
    console.log(cost_data);
  });

  $: {
    if (index) {
    }
    // console.log(index + 1)
    startTime = Date.now();
  }
  // console.log('script end')
</script>

<main>
  <ComputingCostGraph {cost_data} />
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
      class="background"
      slot="background"
      bind:clientWidth={width}
      bind:clientHeight={height}
    >
      <div class="block">
        <div class="col_left"></div>
        <div class="col_right">
          <div class="centering_container">
            {#if index == 0 || index == 1}
              <TitleScreen {index} />
            {/if}
            <!-- {#if index == 1}
              <Graph {data} {index} />
            {/if}
            {#if index == 2}
              <LogGraph {data} {index} />
            {/if} -->
            {#if index == 6}
              <WriteUpScreen />
            {/if}
          </div>
          <div class="progress-bars" hidden>
            <p>current section: <strong>{index + 1}/{count}</strong></p>
            <progress value={count ? (index + 1) / count : 0} />
            <p>offset in current section</p>
            <progress value={offset || 0} />
            <p>total progress</p>
            <progress value={progress || 0} />
          </div>
          <div class="scroll_down">
            <ScrollToContinueIcon {curtime} {count} {index} />
          </div>
        </div>
      </div>
    </div>

    <div class="foreground" slot="foreground">
      <section></section>
      <!-- <section><StoryGraph {data}/></section> -->
      <section><Graph {data} {index} /></section>
      <section><LogGraph {data} {index} /></section>
      <section></section>
      <section></section>
      <section class="gap"></section>
      <section></section>
    </div>
  </Scroller>
</main>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Lato:ital,wght@0,100;0,300;0,400;1,100;1,300;1,400&display=swap");

  /* h1 {
    text-align: center;
    font-family: "Lato", sans-serif;
  } */
  main {
    font-family: "Lato", sans-serif;
  }
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
  }
  .foreground {
    width: 70%;
    margin: 0 auto;
    height: auto;
    position: relative;
  }
  .block {
    display: flex;
    height: 100vh;
  }
  .col_left {
    padding: 20px;
    width: 0%;
    /* If you want to ensure content is vertically aligned, you can add */
    display: flex;
    justify-content: center; /* Center horizontally */
    align-items: center; /* Center vertically */
  }
  .col_right {
    padding: 20px;
    width: 100%;
    height: 92%;
    /* Same vertical alignment styles if needed */
    display: flex;
    flex-flow: column wrap;
    justify-content: center; /* Center horizontally */
    align-items: center; /* Center vertically */
  }
  .centering_container {
    width: 100%;
    height: 80%;
    display: flex;
    justify-content: center; /* Center horizontally */
    align-items: center; /* Center vertically */
  }
  .scroll_down {
    /* height: 100%; */
    display: flex;
    flex-flow: column;
    flex-grow: 1;
    justify-content: flex-end;
  }
  section {
    height: 100vh;
    background-color: rgba(0, 0, 0, 0); /* 20% opaque */
    color: white;
    /* outline: magenta solid 3px; */
    text-align: center;
    max-width: 1000%; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
    align-items: center;

    position: sticky;
    top: 10%;
  }
  section.hidden {
    /* visibility: hidden; */
    animation: fadeOut 1s;
    animation-fill-mode: forwards;
  }
  section.gap {
    height: 10vh;
  }
  @keyframes fadeOut {
  }
</style>
