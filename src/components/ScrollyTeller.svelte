<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import Graph1 from "./Graph.svelte";
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let count, index, offset, progress;
  let width, height;
  let data = [];

  onMount(async () => {
    data = await d3.csv("transistors.csv", (d) => {
      return {
        date: new Date(d.Year),
        value: +d["transistors_per_microprocessor"],
      };
    });
    data = data;
  });
</script>

<main>
  <h1>Moore's Law</h1>
  <Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress
  >
  <div class="background" slot="background" bind:clientWidth={width} bind:clientHeight={height}>
    <Graph1 {data} />
  </div>
  
    <div class="foreground" slot="foreground">
      <section></section>
      <section></section>
      <section></section>
      <section></section>
      <section></section>
      <section></section>
    </div>
  </Scroller>
</main>

<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    /* outline: green solid 3px; */
  }

  .foreground {
    width: 50%;
    margin: 0 auto;
    height: auto;
    position: relative;
    /* outline: red solid 3px; */
  }

  section {
    height: 80vh;
    background-color: rgba(0, 0, 0, 0); /* 20% opaque */
    /* color: white; */
    /* outline: magenta solid 3px; */
    text-align: center;
    max-width: 750px; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }
</style>
