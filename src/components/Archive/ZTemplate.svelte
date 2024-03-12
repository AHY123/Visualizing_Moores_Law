<script>
    import * as d3 from "d3";
    // import { onMount } from "svelte";

    export let index;

    let isVisible;

    $: if (index >= 1 && index <= 3) {
        isVisible = true;
    } else {
        isVisible = false;
    }
    // onMount(() => {

    // })
</script>

<div class='container' class:visible={isVisible}>


</div>

<style>
    .container {
        visibility: hidden;
    }

    .container.visible {
        visibility: visible;
    }
</style>