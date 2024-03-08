<script>
    import * as d3 from "d3";
    // import { onMount } from "svelte";

    export let data;

    let svg, gx, gy;
    let height = 500;
    let width = 800;

    const margin = {
        top: 20,
        right: 30,
        bottom: 50,
        left: 100,
    };

    let isVisible = true;

    // $: if (index >= 1 && index <= 3) {
    //     isVisible = true;
    // } else {
    //     isVisible = false;
    // }
    // onMount(() => {

    // })
</script>

<div class="container" class:visible={isVisible}>
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <svg
      class="graph"
      style="border:5px solid black"
      class:visible={isVisible}
      this="{svg}{width}"
      {height}
      viewBox="0 0 {width} {height}"
    >
    </svg>
</div>

<style>
    .container {
        visibility: hidden;
    }

    .container.visible {
        visibility: visible;
    }
</style>
