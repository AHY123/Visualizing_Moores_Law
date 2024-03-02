<script>
    import * as d3 from "d3";
    // import { onMount } from "svelte";

    export let index;

    let isVisible = false;

    $: if (index == 0) {
        isVisible = false;
    } else {
        isVisible = true;
    }
    // onMount(() => {

    // })
</script>

<div class='container visible' class:visible={isVisible}>
    <h1>
        Visualizing Moore's Law
    </h1>
</div>

<style>
    h1 {
        font-size: 100px;
    }
    .container {
        width: 100%;
        height: 80%;
        display: flex;
        justify-content: center; /* Center horizontally */
        align-items: center; /* Center vertically */
        visibility: visible;
    }
    .container.visible {
        visibility: visible;
        animation: fadeOut 2s ;
        animation-fill-mode: forwards;
    }
    @keyframes fadeOut {
        0% {
            opacity: 1;
        }
        100% {
            opacity: 0;
        }
    }
</style>