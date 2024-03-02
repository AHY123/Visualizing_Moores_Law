<script>
    import * as d3 from "d3";
    // import { onMount } from "svelte";

    export let index;

    let startFade = false;

    $: if (index == 0) {
        startFade = false;
    } else {
        startFade = true;
    }
    // onMount(() => {

    // })
</script>

<div class='container visible' class:fade={startFade}>
    <h1>
        Visualizing Moore's Law
    </h1>
    <h2>
        By Kevin Wong & Andrew H. Yang
    </h2>
</div>

<style>
    h1 {
        font-size: 100px;
    }
    h2 {
        font-size: 30px;
    }
    .container {
        width: 100%;
        height: 80%;
        display: flex;
        flex-flow: column wrap;
        justify-content: center; /* Center horizontally */
        align-items: center; /* Center vertically */
        visibility: visible;
    }
    .container.fade {
        animation: fadeOut 0.5s ;
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