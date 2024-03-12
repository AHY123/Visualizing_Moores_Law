<script>
    import * as d3 from "d3";
    import { fade } from "svelte/transition";

    export let index;
    export let fadeIn;
    export let fadeOut;
    export let title = "";
    export let left = "";
    export let right = "";
    export let left_title = "";
    export let right_title = "";

    let startFadeIn = true;
    let startFadeOut = false;
    let remove = false;

    $: {
        console.log(index)
        if (index > fadeOut) {
            remove = true;
            startFadeIn = false;
            startFadeOut = false;
        }
        else {
            remove = false;
            if (index == fadeIn) {
                console.log('start fade in')
                startFadeIn = true;
                startFadeOut = false;
            }
            if (index == fadeOut) {
                console.log('start fade out')
                startFadeIn = false;
                startFadeOut = true;
            }
        }
    }
</script>

<div class="container" class:fadeIn={startFadeIn} class:fadeOut={startFadeOut} class:remove={remove}>
    <h1>{title}</h1>
    <div class="body">
        <div class="col left">
            <h2>{left_title}</h2>
            <p>{left}</p>
        </div>
        <div class="col mid" />
        <div class="col right">
            <h2>{right_title}</h2>
            <p>{right}</p>
        </div>
    </div>
</div>

<style>
    h1 {
        font-size: 36px;
    }
    h2 {
        font-size: 24px;
        height: 10%;
    }
    p {
        font-size: 18px;
        height: 90%;
    }
    .body {
        display: flex;
        height: 100vh;
        width: 100%;
    }
    .col {
        padding: 20px;
        width: 20%;
        display: flex;
        flex-flow: column;
        justify-content: center;
        align-items: center;
        /* outline: magenta solid 3px; */
    }
    .col.mid {
        width: 60%;
    }
    .container {
        width: 100%;
        height: 100%;
        display: flex;
        flex-flow: column;
        justify-content: center;
        align-items: center;
        visibility: visible;
    }
    .container.fadeIn {
        animation: fadeIn 1s;
        animation-fill-mode: forwards;
    }
    .container.fadeOut {
        animation: fadeOut 1s;
        animation-fill-mode: forwards;
    }
    .container.remove {
        display: none;
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
</style>
